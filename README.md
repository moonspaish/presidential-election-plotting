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
# Step 2: Preparing the election data
## For this part I thankfully found and followed a tutorial made by Raluca Nicola from her Colors of Romania, huge thanks to her. The project she made is one of the only sources where I could find something similar to what I was trying to do. I wasn't familiar with .agg before reading her tutorial so this helped a lot. You can find her project at https://raluca-nicola.net/colors-of-romania .
## 2.1 Grouping and calculating the sum in each UAT 
```
columns = ['a', 'b', 'b1', 'b2', 'b3', 'c', 'd', 'e', 'f',
       'g1', 'g2', 'g3', 'g4', 'g5', 'g6', 'g7', 'g8', 'g9', 'g10', 'g11',
       'g12', 'g13', 'g14'] #choose which columns to sum, each of them have a meaning explained in the document provided with the data
sum_parties = {col: "sum" for col in columns} # here I created a dict which basically has instructions for the .agg function
df2 = df.groupby(by=["Uat", "Județ"]).agg(sum_parties).reset_index()
```
## 2.2 Making a column which specifies the UAT and county and removing words which commonly precede area names and replacing Romanian specific symbols.
```
df2["name_uat"] = (df2["Uat"] + df2["Județ"]).transform(lambda s: s.replace(" ", "").replace("-", "").replace('MUNICIPIUL', '').replace('COMUNA', '').replace('ORAŞ', '').replace("Ă", "A").replace("Ţ", "T").replace("Ă", "A").replace("Ş", "S").replace("Â", "I").replace("Î", "I").lower())

for i in range(1, 7):
  df2["name_uat"]=df2["name_uat"].apply(lambda s: s.replace("bucurestibucurestisector" + str(i), "bucurestisectorul" + str(i) + "bucuresti"))
#correcting certain specific misspellings
df2["name_uat"]=df2["name_uat"].apply(lambda s: s.replace("hirseștiarges", "hirsestiarges"))
df2["name_uat"]=df2["name_uat"].apply(lambda s: s.replace("unousatumare", "orasunousatumare"))
```
# Step 3: Preparing the geographical data
```
uat_localitati='ro_uat_poligon.geojson'
gdf = geopandas.read_file(uat_localitati)
gdf2=gdf.drop(['natcode','natLevName', 'countyId', 'countyCode',
       'countyMn', 'regionId', 'regionCode', 'region', 'pop2011', 'pop2012',
       'pop2013', 'pop2014', 'pop2015', 'pop2020', 'version'], axis=1)
gdf2["name_uat"] = (gdf["name"] + gdf["county"]).transform(lambda s: s.lower().replace(" ", "").replace("-", "").replace("ă", "a").replace("ț", "t").replace("ă", "a").replace("ș", "s").replace("â", "i").replace("î", "i"))
gdf2 = gdf2.sort_values(by="name_uat")
for i in range(1, 7):
  gdf2["name_uat"]=gdf2["name_uat"].apply(lambda s: s.replace("bucurestisector" + str(i), "bucurestisectorul" + str(i) + "bucuresti"))
gdf2["name_uat"]=gdf2["name_uat"].apply(lambda s: s.replace("hirseștiarges", "hirsestiarges"))
gdf2["name_uat"]=gdf2["name_uat"].apply(lambda s: s.replace("orasorasunousatumare", "orasunousatumare"))
```
# Step 4: Merging the election data with the geo data
```
merged3 = df2.merge(gdf2, left_on = "name_uat", right_on = "name_uat", how="outer")
```
# Step 5 Removing data we don't use
## Step 5.1 Removing the votes from the diaspora\
```
merged3=merged3[merged3['geometry'].isnull()==False]
final1=merged3.dropna(subset=['name']) # this is probably useless since name has no null values
```
## Step 5.1 Removing columns which we don't use such as vote presence 
```
final1=final1.drop(['a', 'b', 'b1', 'b2', 'b3', 'c', 'd', 'e', 'f'], axis=1)
```
# Step 6: Naming the columns properly and calculating who is the predominant party, how many votes it has received and what percentage of the total votes it has received.
```
candidati = {'g1': 'Klaus-Werner Iohannis'
,'g2': 'Theodor Paleologu'
,'g3': 'Ilie-Dan Barna'
,'g4': 'Hunor Kelemen'
,'g5': 'Vasilica-Viorica Dancilă'
,'g6': 'Cătălin-Sorin Ivan'
,'g7': 'Ninel Peia'
,'g8': 'Sebastian-Constantin Popescu'
,'g9': 'John-Ion Banu'
,'g10': 'Mircea Diaconu'
,'g11': 'Bogdan-Dragoș-Aureliu Marian-Stanoevici'
,'g12': 'Ramona-Ioana Bruynseels'
,'g13': 'Vioerl Cataramă'
,'g14': 'Alexandru Cumpănașu'}
final1.rename(columns=candidati,
          inplace=True)
columns=['Klaus-Werner Iohannis', 'Theodor Paleologu',
       'Ilie-Dan Barna', 'Hunor Kelemen', 'Vasilica-Viorica Dancilă',
       'Cătălin-Sorin Ivan', 'Ninel Peia', 'Sebastian-Constantin Popescu',
       'John-Ion Banu', 'Mircea Diaconu',
       'Bogdan-Dragoș-Aureliu Marian-Stanoevici', 'Ramona-Ioana Bruynseels',
       'Vioerl Cataramă', 'Alexandru Cumpănașu']

final1["pred_absolute"] = final1[columns].max(axis=1) # the number of the votes the winning party has received
final1["pred_percent"] = round(final1[columns].max(axis=1)/final1[columns].sum(axis=1) * 100, 2)  # the percentage of the vote the winning party has received
final1["pred_party"] = final1[columns].idxmax(axis=1)  # the name of the winning party
final1.drop(['Județ','Uat'], axis=1, inplace=True)     # dropping county and UAT because we can use the column made from their combination, name_uat
final2=final1.dropna()                               
```
# Step 7: Preparing for Plotting 
## Step 7.1 Changing POLYGON geometry to MULTIPOLYGON for plotly to work better or at all ( probably skippable if you don't intend to use plotly )
```
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
gdf4=gdf3
gdf4['geometry']=myl
```
## Step 7.2 Creating Color maps 
```
import matplotlib
from matplotlib.colors import LogNorm 
io_cmap3=['#ffe01a','#e6c700','#806f00']
vi_cmap3=['#f25a5f','#ee2b31','#d41118']
ke_cmap3=['#80cc8c','#5bbd6b','#42a452']
io_cmap5=['#ffe01a','#e6c700','#b39b00','#806f00','#4d4200']
vi_cmap5=['#ee2b31','#d41118','#a50d13','#760a0d','#470608']
ke_cmap5=['#5bbd6b','#42a452','#337f40','#255b2d','#16371b']

cmap1 = matplotlib.colors.ListedColormap(io_cmap3)
cmap2 = matplotlib.colors.ListedColormap(vi_cmap3)
cmap3 = matplotlib.colors.ListedColormap(ke_cmap3)
cmap4 = matplotlib.colors.ListedColormap(io_cmap5)
cmap5 = matplotlib.colors.ListedColormap(vi_cmap5)
cmap6 = matplotlib.colors.ListedColormap(ke_cmap5)
```
# Step 8: Plotting
## Matplotlib one color for each candidate
```
from matplotlib.colors import LogNorm 
import matplotlib
colors =["lightblue","green" , "yellow",'blue','purple',"red"]
cmap = matplotlib.colors.ListedColormap(colors)
# ax = gdf4.plot(
#     column="pred_party",
#     k=4,
#     cmap=cmap,
#     legend=True,
#     legend_kwds={"loc": "center left", "bbox_to_anchor": (1, 0.5)},
#     figsize=(15, 10)
# )
# ax.set_axis_off()
# ax
```
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/2c96e616-b9d7-4671-a746-0686328c97fe)
## Matplotlib 5 colors for each of the three main candidates
```
import matplotlib.colors as colors
idk=gdf4[gdf4["pred_party"]=="Klaus-Werner Iohannis"]
ax = idk.plot(
    column="pred_percent",
    cmap=cmap4,
    scheme='equalinterval',
    k=5,
    figsize=(15, 10)
)
idk2=gdf4[gdf4["pred_party"]=='Vasilica-Viorica Dancilă']
idk2.plot( ax=ax,
    column="pred_percent",
    cmap=cmap5,
    scheme='equalinterval',
    k=5,
)
idk3=gdf4[gdf4["pred_party"]=='Hunor Kelemen']
idk3.plot( ax=ax,
    column="pred_percent",
    cmap=cmap6,
    scheme='equalinterval',
    k=5
)
idk4=gdf4[~gdf4["pred_party"].isin(['Hunor Kelemen','Vasilica-Viorica Dancilă',"Klaus-Werner Iohannis"])]
idk4.plot( ax=ax,
    column="pred_percent",
    cmap='Blues',
    scheme='equalinterval',
    k=5
)
ax.set_axis_off()
ax
```
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/5a9ca647-bd53-4178-b2c5-aa41a745176b)
## Folium code
```
idk_1=gdf4[gdf4["pred_party"]=="Klaus-Werner Iohannis"]
idk_dict=idk.to_json(drop_id=True)
idk_2=gdf4[gdf4["pred_party"]=='Vasilica-Viorica Dancilă']
idk2_dict=idk2.to_json(drop_id=True)
idk_3=gdf4[gdf4["pred_party"]=='Hunor Kelemen']
idk3_dict=idk3.to_json(drop_id=True)
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
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/7a3b7213-19b1-4186-b1e3-44f06d5fca36)
## Plotly code 
```
token = #put your key here
fig = px.choropleth_mapbox(gdf3, geojson=gdf3.geometry, color=gdf3['pred_party'],
                    locations=gdf3.index,labels=gdf3["Județ"],hover_name=gdf3['pred_absolute'],
                    color_discrete_sequence=['yellow','red',"green"], height=800, width=1200,
                    mapbox_style = 'open-street-map',center = dict(lat = 45.9432, lon = 24.9668),
                    zoom = 6)
fig.update_traces(marker_line_width = 0.1, marker_line_color = 'black')
fig.update_geos(fitbounds="geojson", visible=False)
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.update_layout(mapbox_style="outdoors", mapbox_accesstoken=token,
                  mapbox_zoom=6, mapbox_center = {"lat": 45.9432, "lon": 24.9668})
fig.show()
```
![image](https://github.com/moonspaish/presidential-election-plotting/assets/69521713/a430533b-6a24-44d1-90d4-5245bad4b6d8)

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

