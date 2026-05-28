# Land Surface Temperature of Kolkata

Estimation of Land Surface Temperature of Kolkata in Celsius using Landsat 8 satellite imagery processed with Google Earth Engine.

## Overview

This project calculates the Land Surface Temperature (LST) for Kolkata, West Bengal, India using Landsat 8 thermal band data. The analysis covers the period from March 1 to May 1, 2024.

## Temperature Range

| Metric | Temperature |
|--------|-------------|
| **Highest Temperature** | 37°C |
| **Lowest Temperature** | 27°C |

## Methodology

The LST calculation follows these steps:
1. **Data Collection**: Landsat 8 imagery filtered for Kolkata district (2024-03-01 to 2024-05-01)
2. **NDVI Calculation**: Normalized Difference Vegetation Index from RED and NIR bands
3. **Thermal Radiometry**: Spectral radiance conversion from thermal band (B10)
4. **Brightness Temperature**: Conversion using Planck's Law constants
5. **LST Calculation**: Final LST computation accounting for emissivity
6. **Unit Conversion**: Conversion from Kelvin to Celsius

## Code

```javascript
var Kolkata = table.filter(ee.Filter.eq("District","KOLK>TA"));
var landsat8 = ee.ImageCollection("LANDSAT/LC08/C02/T1")
.filterDate("2024-03-01","2024-05-01")
.filterBounds(Kolkata);
print("Metadata",landsat8);
var medianImage = landsat8.median();
var clippedImage = medianImage.clip(Kolkata);
var visLandsat8 = {
  bands:["B4","B3","B2"],
  min:9217.5,
  max:12664,
  gamma:1.0
};
var NDVI = clippedImage.expression (
  "(NIR-RED)/(NIR+RED)",
  {
    "NIR":clippedImage.select("B5"),
    "RED":clippedImage.select("B4")
  }
);
var visNDVI = {
  min:-0.08988826505086442,
  max:0.3727009342752043,
  gamma:1.0
};
var l = clippedImage.expression (
  "ML*Qcal+AL",
  {
    "ML":0.0003342,
    "Qcal":clippedImage.select("B10"),
    "AL":0.1
  }
);
var Pv = NDVI.expression (
  "(NDVI-NDVImin)/(NDVImax-NDVImin)**2",
  {
    "NDVI":NDVI,
    "NDVImin":-0.08988826505086442,
    "NDVImax":0.3727009342752043
  }
);
var e = Pv.expression (
  "0.004*Pv+0.986",
  {
    "Pv":Pv
  }
);
var BT = l.expression (
  "K2/log((K1/l)+1)",
  {
    "K1":774.8853,
    "K2":1321.0789,
    "l":l
  }
);
var lst = BT.expression (
  "BT/(1+((lambda*BT /p)*log(e)))",
  {
    "BT":BT,
    "lambda":10.895e-6,
    "p":1.438e-2,
    "e":e
  }
);
var Celcius = lst.expression (
  "lst-273.15",
  {
    "lst":lst
  }
);
var visCelcius = {
  min:27.517696590087496,
  max:37.1750714051509,
  palette:["#000000","#170000","#2d0000","#440000","#5a0000",
           "#9f0000","#d83c16","#f77a2c","#f9e317","#f9f7d4"]
};
Map.addLayer(Celcius,visCelcius);
Map.centerObject(Kolkata,12);
var legend = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '8px 15px'
  }
});
var legendTitle = ui.Label({
  value: 'LEGEND',
  style: {
    fontFamily: 'calibri',
    fontWeight: 'bold',
    fontSize: '20px',
    margin: '0 0px 0',
    padding: '0'
  }
});
legend.add(legendTitle);
var legend1 = ui.Panel({
  style: {
    position: 'top-center',
    padding: '8px 15px'
  }
});
var legendTitle1 = ui.Label({
  value: "Land Surface Temperature in Centigrate (°C), Kolkata, West Bengal,India",
  style: {
    fontFamily: 'calibri',
    fontWeight: 'bold',
    fontSize: '20px',
    margin: '0 0px 0',
    padding: '0'
  }
});
legend1.add(legendTitle1);
var palette = ["#f9f7d4","#f9e317","#f77a2c","#d83c16","#9f0000",
               "#5a0000","#440000","#2d0000","#170000","#000000"
];
var names = ["37°C (Highest Temperature)","","","","",
             "","","","","27°C (Lowest Temperature)"
];
for (var i = 0; i < palette.length; i++) {
  var colorBox = ui.Label({
    style: {
      backgroundColor: palette[i],
      padding: '6px 9px',
      margin: '0px 5px 0px 0px',
    }
  });
  var description = ui.Label({
    value: names[i],
    style: {
      margin: '1px 1px 1px 1px',
      fontFamily:"arial",
      fontSize:"13px",
      fontWeight:"bold"
    }
  });
  var legendItems = ui.Panel({
    widgets: [colorBox, description],
    layout: ui.Panel.Layout.Flow('horizontal')
  });
  legend.add(legendItems);
}
Map.add(legend);
Map.add(legend1);
Map.addLayer(Kolkata.style({fillColor:"00000000",color:"white",width:2.5}),{});
Export.image.toDrive({
  image: Celcius,
  description: 'LST_Landsat8_Kolkata',
  folder: 'GEE_Exports',         // Create in Google Drive or leave blank
  fileNamePrefix: 'LST_Landsat8_Kolkata',
  region: Kolkata,
  scale: 30,                     // Landsat resolution
  crs: 'EPSG:4326',
  maxPixels: 1e13
});
```

## Output

The script generates:
- LST visualization layer with color gradient mapping
- Interactive legend showing temperature range
- Exported GeoTIFF file to Google Drive (30m resolution)
- Filename: `LST_Landsat8_Kolkata`

## Platform

- **Google Earth Engine** - For satellite data processing and analysis
- **Landsat 8** - Thermal infrared data source
