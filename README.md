## Overview
<p align="justify">This project aims to address the critical issue of food insecurity within national Medicare Advantage plans by deploying a targeted food access program. Leveraging publicly available data from the CDC's 500 Cities Project and the FDA's Food Atlas, the analysis identifies areas with high poverty rates and food insecurity to optimize program impact.</p>

## Interactive Visualization
To view the interactive visualizations along with the entire code, please visit [nbviewer](https://nbviewer.org/github/arnavv-agarwal/Food-Access-Analysis/blob/main/Food_Access_Analysis.ipynb).

<p align="justify">Through this analysis, we anticipate a significant reduction in food insecurity, low access to stores, and poverty rates in targeted regions, thereby enhancing the well-being of Medicare Advantage plan members.</p>

## Objective
<p align="justify">The primary objective is to provide actionable insights for deploying a food access program that maximizes impact on the Medicare Advantage plan's membership. This involves determining deployment locations, estimating program reach, identifying target populations, and projecting the program's impact.</p>

## Features
- Utilization of CDC's 500 Cities Project and FDA's Food Atlas data.
- Identification of counties with above-average poverty rates and states with high food insecurity.
- Visualizations to highlight areas in need of a food access program.
- Estimation of program coverage and potential reach.
- Identification of population segments benefiting the most from the program.
- Projection of the program's impact on food insecurity, access to stores, and poverty rates.

## Technology Stack
- Python for data analysis and visualization (using libraries like Pandas, Matplotlib, and Seaborn).
- Folium for spatial data analysis.
- Pandas for data aggregation and manipulation.

## Data
- CDC - 500 Cities Project: Provides census tract-level data on various health measures.
- FDA - Food Atlas: Offers more than 280 variables related to food environment, including food banks and nutrition assistance program participation rates.

## Assumptions
- Deployment areas identified based on above-average poverty rates and food insecurity.
- Segmentation of population into those included in the program and those potentially engaged.
- Priority given to segments exceeding national poverty rates and facing food insecurity.
- Projected impact includes decreases in food insecurity, low access to stores, and poverty rates following program deployment.

## Insights
- Where should we deploy a food access program?
  - We should focus on counties with average poverty rate of over 11.6% (National Average)
  - We should also consider simultaneously the states which have over 10.2% of food insecutiry (National Average)

```python 
for index, row in cdc_fda_main_deploy_filtered.iterrows():
    folium.CircleMarker(
        location=[row['latitude'], row['longitude']],
        radius=row['MA_POP'] / population_divisor,  
        color='blue',
        fill=True,
        fill_color='blue',
        fill_opacity=0.6,
        tooltip=f"Neighborhood: {row['StateAbbr']}<br>MA Population: {row['MA_POP']}"
    ).add_to(marker_cluster)
```
```python 
map_obj.save("food_access_map.html")
map_obj
```
<p align="center">
  <img width="500" height="300" src="https://github.com/arnavv-agarwal/Food-Access-Analysis/blob/main/FA1.png">
</p>

- How many people will be included? How many might be successfully engaged?
  - <p align="justify">The blue color represents the population that will be included in the program, while the red color represents the population that has the potential to engage with the program. This differentiation allows us to visually understand the overlap and distinction between the two groups, providing valuable insights into program coverage and potential reach.</p>
```python 
# Calculate the total MA_POP_FInsec and total Eng_Pop for each state
state_totals = cdc_fda_main_deploy.groupby('StateAbbr').agg({'MA_POP_FInsec': 'sum','Eng_Pop': 'sum'}).reset_index()

# Create a bar chart to visualize the total MA_POP_FInsec and total Eng_Pop for each state
fig = px.bar(state_totals, x='StateAbbr', y=['MA_POP_FInsec', 'Eng_Pop'],
             title='Total Population Included and Engaged (Population by State)',
             labels={'value': 'Population', 'variable': 'Population Type', 'StateAbbr': 'States'},
             barmode='group')

# Display the chart
fig.show()
```

<p align="center">
  <img width="800" height="400" src="https://github.com/arnavv-agarwal/Food-Access-Analysis/blob/main/FA2.png">
</p>

- Which subgroup of the population might benefit the most from the program?
  - <p align="justify">In the below visualization, the red color represents a specific segment of the engaged population that would benefit the most from the food access program. This segment comprises individuals who exceed the national poverty rate and face food insecurity. Furthermore, this population subset also includes individuals with low income levels and limited access to stores. By highlighting this group, we can identify and prioritize the individuals who are most in need of support and intervention from the food access program, addressing their specific challenges related to poverty, food insecurity, and limited access to stores.</p>
```python 
# Calculate the total MA_POP_FInsec and total Eng_Pop for each state
state_totals = cdc_fda_main_deploy.groupby('StateAbbr').agg({'Eng_Pop': 'sum', 'MA_POP_LILACC': 'sum'}).reset_index()

# Create a bar chart to visualize the total MA_POP_FInsec and total Eng_Pop for each state
fig = px.bar(state_totals, x='StateAbbr', y=['Eng_Pop', 'MA_POP_LILACC'],
             title='Most Benefitted Subgroup of Population ',
             labels={'value': 'Population', 'variable': 'Population Type', 'StateAbbr': 'States'},
             barmode='group')

# Display the chart
fig.show()
```
<p align="center">
  <img width="800" height="400" src="https://github.com/arnavv-agarwal/Food-Access-Analysis/blob/main/FA3.png">
</p>

- What is the projected impact of this program?
  - To assess the success of the Food Access Program, we will focus on three key factors:
      - Food Insecurity
      - Low Access to Stores
      - Poverty Rate
```python
# Create temporary dataset
cdc_fda_prj_imp = cdc_fda_main_deploy[['StateAbbr', 'FOODINSEC_15_17', 'PCT_LACCESS_POP15', 'POVRATE15', 'Population2017', 'Eng_Pop']]

grouped_cdc_fda_prj_imp = cdc_fda_prj_imp.groupby('StateAbbr').agg({'Population2017': 'sum','FOODINSEC_15_17': 'mean','PCT_LACCESS_POP15': 'mean','POVRATE15': 'mean', 'Eng_Pop' : 'sum'}).reset_index()

# Create necessary columns 
grouped_cdc_fda_prj_imp['FOODINSEC_Pop'] = (grouped_cdc_fda_prj_imp['Population2017']*(grouped_cdc_fda_prj_imp['FOODINSEC_15_17']/100)).astype(int)
grouped_cdc_fda_prj_imp['LACCESS_Pop'] = (grouped_cdc_fda_prj_imp['Population2017']*(grouped_cdc_fda_prj_imp['PCT_LACCESS_POP15']/100)).astype(int)
grouped_cdc_fda_prj_imp['POV_Pop'] = (grouped_cdc_fda_prj_imp['Population2017']*(grouped_cdc_fda_prj_imp['POVRATE15']/100)).astype(int)

grouped_cdc_fda_prj_imp['FOODINSE_Ben_Pop'] = (grouped_cdc_fda_prj_imp['FOODINSEC_Pop']- grouped_cdc_fda_prj_imp['Eng_Pop']).astype(int)
grouped_cdc_fda_prj_imp['LACCESS_Ben_Pop'] = (grouped_cdc_fda_prj_imp['LACCESS_Pop']- grouped_cdc_fda_prj_imp['Eng_Pop']).astype(int)
grouped_cdc_fda_prj_imp['POV_Ben_Pop'] = (grouped_cdc_fda_prj_imp['POV_Pop']- grouped_cdc_fda_prj_imp['Eng_Pop']).astype(int)

grouped_cdc_fda_prj_imp['Proj_FOODINSEC'] = (grouped_cdc_fda_prj_imp['FOODINSE_Ben_Pop']/grouped_cdc_fda_prj_imp['Population2017'])*100
grouped_cdc_fda_prj_imp['Proj_LACCESS'] = (grouped_cdc_fda_prj_imp['LACCESS_Ben_Pop']/grouped_cdc_fda_prj_imp['Population2017'])*100
grouped_cdc_fda_prj_imp['Proj_POV'] = (grouped_cdc_fda_prj_imp['POV_Ben_Pop']/grouped_cdc_fda_prj_imp['Population2017'])*100
```
> [!IMPORTANT]  
> From the above analysis, the following changes are evident after the successful deployment of the Food Access Program in the targeted regions:
- Approximately 4.3679046721273735% decrease in Food Insecurity
- Approximately 4.367881441510681% decrease in Population with Low Access to Stores
- Approximately 4.367932977411778% decrease in Poverty Rate
