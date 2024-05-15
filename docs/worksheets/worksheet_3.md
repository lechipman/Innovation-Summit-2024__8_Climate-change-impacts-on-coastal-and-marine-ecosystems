# TEAM ACTIVITY 2: Innovate as a Team

Welcome back! We hope today is a productive day getting to know your team and coding.

## Day 2 summary: 
Please complete the warm-up with your team, briefly review today’s objectives, and carefully read the Day 2 and Day 3 report out items to guide your efforts.  

## Objectives for Day 2
1. Work together to decide on the data sets you will use. Reminder: Use a decision-making technique discussed during Day 1. Kaner’s Gradient of Agreement is below for reference.
2. Practice joining your datasets together. 
3. Discuss and try creating interesting graphics.
4. Report back on your results at the end of the day. Today’s report back is short and focused on your team process. The Day 3 report back is more detailed. 


## Morning Warm-up
Please share the following informaton with your team. (No need to write down your responses this time)
- Name
- Pronouns
- Reflecting on Day 1, what is something that surprised you?

## Decision-Making
Use the gradient of agreement (Kaner 20214) to make decisions as a team.

![Gradients of agreement](../worksheets/love_gradient-of-agreement.png)



``` python
import requests
import geopandas as gpd
import matplotlib.pyplot as plt
from io import BytesIO
import zipfile
import os

# Step 1: Download the data
url = 'https://chs.coast.noaa.gov/htdata/Inundation/SLR/SLRdata/SC/SC_South_slr_data_dist.zip'
response = requests.get(url)

# Check if the response is successful
if response.status_code == 200:
    try:
        with zipfile.ZipFile(BytesIO(response.content)) as z:
            z.extractall('sea_level_data')
    except zipfile.BadZipFile:
        print("Error: The downloaded file is not a valid ZIP file.")
else:
    print(f"Error: Unable to download the file. Status code: {response.status_code}")

# Step 2: Load the data from the geodatabase
gdb_path = 'sea_level_data/SC_South_slr_final_dist.gdb'

try:
    # List all layers in the geodatabase
    layers = gpd.io.file.fiona.listlayers(gdb_path)
    print(f"Available layers: {layers}")

    # Read the desired layer (replace 'SC_South_slr_3ft' with the desired layer name)
    layer_name = 'SC_South_slr_3ft'
    gdf = gpd.read_file(gdb_path, layer=layer_name)
    
    # Print column names to inspect
    print(f"Columns in layer '{layer_name}': {gdf.columns}")
except Exception as e:
    print(f"Error loading geodatabase: {e}")
    raise

# Step 3: Filter data for the southeastern US (if needed)
try:
    # You can filter based on bounding box, state names, etc.
    # For example, filter for South Carolina
    bbox = [-83.0, 32.0, -78.0, 35.0]  # Bounding box for South Carolina
    southeast_us_gdf = gdf.cx[bbox[0]:bbox[2], bbox[1]:bbox[3]]
except Exception as e:
    print(f"Error filtering data: {e}")
    raise

# Step 4: Plot the data
try:
    fig, ax = plt.subplots(1, 1, figsize=(10, 10))
    southeast_us_gdf.plot(ax=ax, column='gridcode', legend=True,  # Replace 'gridcode' with the correct column name
                          legend_kwds={'label': "Sea Level Rise (meters)"})
    plt.title(f'Projected Sea Level Rise in South Carolina - Layer: {layer_name}')
    plt.xlabel('Longitude')
    plt.ylabel('Latitude')
    plt.show()
except Exception as e:
    print(f"Error plotting data: {e}")
    raise
```

```python
# Mounting to MODIS land cover data and clipping it to bounding box (South Carolina)
import numpy as np
from osgeo import gdal, osr
from shapely.geometry import box
import matplotlib.pyplot as plt
%matplotlib inline

# First define the Cloud-Optimized-GeoTiff URL and create a virtual mount to it with gdal Open
url = 'https://s3.openlandmap.org/arco/lc_mcd12q1v061.t1_c_500m_s_20210101_20211231_go_epsg.4326_v20230818.tif'
dataset = gdal.Open(url)

# Find the bounds that will clip the data to South Carolina
bbox = [-83.0, 32.0, -78.0, 35.0]

# Open the GDAL dataset
dataset = gdal.Open(url)

# Get the dataset's projection and geo-transform
projection = dataset.GetProjection()
geo_transform = dataset.GetGeoTransform()

# Create spatial reference objects for transformations
src_srs = osr.SpatialReference()
src_srs.ImportFromWkt(projection)
dst_srs = osr.SpatialReference()
dst_srs.ImportFromEPSG(4326)  # EPSG:4326 is WGS84, the coordinate system of the bounding box

# Create a transformation object
transform = osr.CoordinateTransformation(dst_srs, src_srs)

# Transform the bounding box coordinates to the dataset's coordinate system
min_x, min_y, _ = transform.TransformPoint(bbox[0], bbox[1])
max_x, max_y, _ = transform.TransformPoint(bbox[2], bbox[3])

# Calculate pixel coordinates
inv_geo_transform = gdal.InvGeoTransform(geo_transform)
min_pixel_x, min_pixel_y = gdal.ApplyGeoTransform(inv_geo_transform, min_x, min_y)
max_pixel_x, max_pixel_y = gdal.ApplyGeoTransform(inv_geo_transform, max_x, max_y)

# Ensure pixel coordinates are within the dataset bounds
min_pixel_x = max(0, int(min_pixel_x))
min_pixel_y = max(0, int(min_pixel_y))
max_pixel_x = min(dataset.RasterXSize, int(max_pixel_x))
max_pixel_y = min(dataset.RasterYSize, int(max_pixel_y))

# Read in the land cover type array and clip it to bounds and plot it
band = dataset.GetRasterBand(1)
array = band.ReadAsArray()
clipped_array = array[max_pixel_y:min_pixel_y, min_pixel_x:max_pixel_x]
plt.imshow(clipped_array)
```


## Day 2 Report Back
Day 2 report-back questions are about the team *process*. We are interested in your team’s unique experience. Below are some prompts you might consider. You don't need to address all of them - choose which ones you want to present. Please limit your reflection to 2-3 mins.  

1. **What worked well for your team?**
3. **What’s one thing you would change?**
4. **Did your group ever have an “ah-ha” moment?  What led up to that moment?**
5. **Did your group experience the groan zone?  What is one tip you want to share with future groups at the Summit about getting through the groan zone?**
     - [insert your group reflection responses here]

**************************************************************

### Looking Ahead: Day 3 Report Back
*These are the prompts for the final Report Back **tomorrow (Day 3)** - start thinking about these questions as you work today. Each group will share their Day 3 GitHub page on the screen and give a 4 minute presentation.*

- **Project Title:**
- **Research Question:**
- **One interesting graphic/finding:**
- **What are you thinking about doing next with your team? Long-term, short-term?**
- **What’s missing: what resources, people, data sets, etc. does your team need?**
      
### Reminder
There is the opportunity for groups to continue working on their projects as an ESIIL Working Group. If you love your team and want to continue working together, considering submitting a Working Group Application this fall. See the ESIIL website for more information: <https://esiil.org/working-groups>.
     

