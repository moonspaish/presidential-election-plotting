# 2019 Romanian Presidential Election plotting 
## Choropleths made using matplotlib 
### Predominant party in each UAT ( Teritorial administrative unit )
```
from matplotlib.colors import LogNorm 
import matplotlib
colors =["lightblue","green" , "yellow",'blue','purple',"red"]
cmap = matplotlib.colors.ListedColormap(colors)
ax = gdf4.plot(
    column="pred_party",
    k=4,
    cmap=cmap,
    legend=True,
    legend_kwds={"loc": "center left", "bbox_to_anchor": (1, 0.5)},
    figsize=(15, 10)
)
ax.set_axis_off()
ax
```

![output simplu](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/33e310e9-ad56-47a5-bb0d-d5e397ec1961)
```
idk=gdf4[gdf4["pred_party"]=="Klaus-Werner Iohannis"]
ax = idk.plot(
    column="pred_percent",
    cmap=cmap4,
    scheme='equalinterval',
    k=3,
    legend=True,
    legend_kwds={"loc": "center left", "bbox_to_anchor": (1, 0.5)},
    figsize=(15, 10)
)
idk2=gdf4[gdf4["pred_party"]=='Vasilica-Viorica Dancilă']
idk2.plot( ax=ax,
    column="pred_percent",
    cmap=cmap5,
    scheme='equalinterval',
    k=3
)
idk3=gdf4[gdf4["pred_party"]=='Hunor Kelemen']
idk3.plot( ax=ax,
    column="pred_percent",
    cmap=cmap6,
    scheme='equalinterval',
    k=3
)
idk4=gdf4[~gdf4["pred_party"].isin(['Hunor Kelemen','Vasilica-Viorica Dancilă',"Klaus-Werner Iohannis"])]
idk4.plot( ax=ax,
    column="pred_percent",
    cmap='Blues',
    scheme='equalinterval',
    k=3
)
ax.set_axis_off()
ax
```
## Predominant party in each UAT but with 5 colors for each of the main cadidates in equal intervals ( Teritorial administrative unit ) 
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/93635761-dbe9-42f5-bebd-4a780975b066)
In my mind this would be ideal with the way folium legends are displayed, using branca colormaps
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/ed3437b0-17dc-4e27-96f7-7e23062296d3)
### But I am pretty certain it is impossible to use branca colourmaps as legends for matplotlib plots so just in case I literally edited a picture to explain what I was going for I am aware this is petty and stupid.
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/494fb13b-ec9e-4ba7-8fa3-de6e83f57f35)
## Choropleth made using folium
```
import folium

m = folium.Map(location=[45.9432,24.9668], tiles="OpenStreetMap", zoom_start=7)

folium.Choropleth(
    geo_data=idk_dict,
    name="choropleth",
    data=idk,
    columns=["name_uat", "pred_percent"],
    key_on="feature.properties.name_uat",
    fill_color='Purples',
    fill_opacity=0.7,
    line_opacity=0.5,
    legend_name='Klaus-Werner Iohannis'
).add_to(m)
folium.Choropleth(
    geo_data=idk2_dict,
    name="choropleth",
    data=idk2,
    columns=["name_uat", "pred_percent"],
    key_on="feature.properties.name_uat",
    fill_color='OrRd',
    fill_opacity=0.7,
    line_opacity=0.5,
    legend_name='Vasilica-Viorica Dancilă'
).add_to(m)
folium.Choropleth(
    geo_data=idk3_dict,
    name="choropleth",
    data=idk3,
    columns=["name_uat", "pred_percent"],
    key_on="feature.properties.name_uat",
    fill_color='YlGn',
    fill_opacity=0.7,
    line_opacity=0.5,
    legend_name='Hunor Kelemen'
).add_to(m)
folium.LayerControl().add_to(m)
m
```

This is the same dataframe but displayed using folium. Which was more confusing than I expected, you can find details in the "Small blog break" part of the post if you hapened to encounter the same issues as I did.
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/a0baf83a-a0b0-4448-9728-3be3b5cb7c05)
## Choropleth using plotly displaying the predominant party by county
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/141aae42-4400-4943-866d-6bda62c3c190)
# Step 1: Getting the data
## Election data:
### The data can be downloaded at the link below and contains the votes, and a file explaining the votes, the data comes from the Permanent Electoral Authority which is a government agency which oversees the electoral infrastructure. 
http://alegeri.roaep.ro/wp-content/uploads/2022/01/AlegeriPrezidentiale2019.zip
### The UAT geojson can be downloaded at the link below, the data comes from geo-spatial.org which as the name implies is an organization which publishes articles, tutorials, but also offers access to data with an open source approach.
https://geo-spatial.org/vechi/file_download/29535







# Small blog break
## This is if anyone tries to do something similar
### The problems I encountered along the way were:
VSCode Jupyter Notebooks crashing - Sollution : In my case I was keeping too many large jsons in memory, slicing them up, keeping what is stored on my RAM to a minimum solved my issues.
Folium not displaying - Sollution : Apparently I was unable to use the same geodataframe to display both the data and the geometries so I was required to create jsons from the dataframes and to call them in order to display the geometries. The way folium works it looks to match the data from the dataframe and the json. Also when converting geopandas df to jsons you might want to try dropping the id if it doesn't work.
### As you can see the converted df into json as the first argument, the df as the third, the columns with an id for the data followed up by the column with the data, and the location of the matching id in the json.
```
    geo_data=idk3_dict,
    name="choropleth",
    data=idk3,
    columns=["name_uat", "pred_percent"],
    key_on="feature.properties.name_uat"
```
### Example which shows the code for both of the previous problems : 
```
idk_1=gdf4[gdf4["pred_party"]=="Klaus-Werner Iohannis"]
idk_dict=idk.to_json(drop_id=True)
idk_2=gdf4[gdf4["pred_party"]=='Vasilica-Viorica Dancilă']
idk2_dict=idk2.to_json(drop_id=True)
idk_3=gdf4[gdf4["pred_party"]=='Hunor Kelemen']
idk3_dict=idk3.to_json(drop_id=True)
```
## Plotly not accepting my geodataframes - Sollution : It seems using geopandas the coordinates were converted into both MULTIPOLYGON and POLYGON geometry types so I needed to convert all tbe coordinates I had stored as POLYGON in to MULTIPOLYGON in order for the code to run.
### Example for how I converted  POLYGON into MULTIPOLYGON:
```
gdf3 = geopandas.GeoDataFrame(final2, geometry="geometry")
from shapely.geometry.multipolygon import MultiPolygon
from shapely import wkt
from shapely.geometry import shape
b=gdf3['geometry'].tolist()
myl=[]
for item in range(0,len(b)):
    if b[item].geom_type=='MultiPolygon':
         myl.append(b[item])
    else:
        myl.append(MultiPolygon([b[item]]))
```

