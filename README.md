## Creating interactive dashboards with Python | Plotly | Dash | IBM Cloud IDE

### Project Goal
Analyze historic tends in automobile sales during recession periods. 
This was the final assignment in module 7 associated with IBM Data Analyst Professional certificate.

### Python Commands Explored
- Create Dropdown menus
- HTML layout
- Callback functions
- Creating and displaying 4 interactive plots

```python
#!/usr/bin/env python

# coding: utf-8

# In[ ]:

import dash

from dash import dcc

from dash import html

from dash.dependencies import Input, Output

import pandas as pd

import plotly.graph_objs as go

import plotly.express as px
```
  

### Load the data using pandas
```python
data = pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/historical_automobile_sales.csv')

  ```

### Initialize the Dash app
```python
app = dash.Dash(__name__)
year_list = [i for i in  range(1980, 2024, 1)]

```
### Create the layout of the app
```python
app.layout = html.Div([

#Add title to the dashboard

html.H1("Automobile Statistics Dashboard",

style={'textAlign': 'center', 'color': '#503D36', 'font-size': 24}),

html.Div([#TASK 2.2: Add two dropdown menus

html.Label("Select Statistics:"),

dcc.Dropdown(

id='dropdown-statistics',

options=[

{'label': 'Yearly Statistics', 'value': 'Yearly Statistics'},

{'label': 'Recession Period Statistics', 'value': 'Recession Period Statistics'}

],

value='Yearly Statistics',

placeholder='Select Statistics',

style={

'textAlign': 'center', 'width': '80%', 'padding': '3px',

'font': '20px Arial, Helvetica, sans-serif'}

)

]),

html.Div(dcc.Dropdown(

id='select-year',

options=[{'label': i, 'value': i} for i in year_list],

value='1980'

)),

html.Div([#TASK 2.3: Add a division for output display

html.Div(id='output-cotainer', className='chart-grid', style={'display': 'flex'}),])

])
```
### Creating Callbacks
Define the callback function to update the input container based on the selected statistics
```python
@app.callback(

Output(component_id='select-year', component_property='disabled'),

Input(component_id='dropdown-statistics',component_property='value'))

  
def  update_input_container(dropdown_statistics):

if selected_statistics =='Yearly Statistics':

return  False

else:

return  True
```
  
### Callback for plotting
Define the callback function to update the input container based on the selected statistics
```python
@app.callback(

Output(component_id='output-container', component_property='children'),

[Input(component_id='dropdown-statistics', component_property='value'), Input(component_id='select-year', component_property='value')])

  
  

def  update_output_container(selected_statistics, input_year):

if selected_statistics == 'Recession Period Statistics':
```
### Filter the data for recession periods
```python
recession_data = data[data['Recession'] == 1]
```
### Create and display graphs for Recession Report Statistics

Plot 1 Automobile sales fluctuate over Recession Period (year wise), use groupby to create relevant data for plotting.
```python
yearly_rec=recession_data.groupby('Year')['Automobile_Sales'].mean().reset_index()

R_chart1 = dcc.Graph(

figure=px.line(yearly_rec,

x='Year',

y='Automobile_Sales',

title="Average Automobile Sales fluctuation over Recession Period"))
```
Plot 2 Calculate the average number of vehicles sold by vehicle type
```python
average_sales = recession_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()

R_chart2 = dcc.Graph(

figure=px.line(average_sales,

x='Automobile_Sales',

y='Vehicle_Type',

title="Average number of vehicles sold by vehicle type"))
```
Plot 3 Pie chart for total expenditure share by vehicle type during recessions
```python
exp_rec= recession_data.groupby('Vehicle_Type')['Advertising_Expenditure'].mean().reset_index()

R_chart3 = dcc.Graph(

figure=px.pie(exp_rec,

values='Advertising_Expenditure',

names='Vehicle_Type',

title="Total expenditure share by vehicle type during recessions"))
```
Plot 4 bar chart for the effect of unemployment rate on vehicle type and sales
```python
unemployment_sales= recession_data.groupby(['Vehicle_Type', 'unemployment_rate'])['Automobile_Sales'].mean().reset_index()

R_chart4 = dcc.Graph(

figure=px.bar(unemployment_sales,

x='unemployment_rate',

y='Automobile_Sales',

color='Vehicle_Type',

title="The effect of unemployment rate on vehicle type and sales"))

return [

html.Div(className='chart-item', children=[html.Div(children=R_chart1),html.Div(children=R_chart1)],style={'display': 'flex'}),

html.Div(className='chart-item', children=[html.Div(children=R_chart2),html.Div(children=R_chart2)],style={'display': 'flex'}),

html.Div(className='chart-item', children=[html.Div(children=R_chart3),html.Div(children=R_chart3)],style={'display': 'flex'}),

html.Div(className='chart-item', children=[html.Div(children=R_chart4),html.Div(children=R_chart4)],style={'display': 'flex'})

]
```
  

### Create and display graphs for Yearly Report Statistics
Yearly Statistic Report Plots
```python
elif (input_year and selected_statistics=='Yearly_Statistics') :

yearly_data = data[data['Year'] == input_year]
```
Creating Graphs Yearly data
plot 1 Yearly Automobile sales using line chart for the whole period.
```python
yas= data.groupby('Year')['Automobile_Sales'].mean().reset_index()

Y_chart1 = dcc.Graph(figure=px.line(yas,

x='Year',

y='Automobile_Sales',

title ='Yearly Automobile sales'))
```
Plot 2 Total Monthly Automobile sales using line chart.
```python
mas= data.groupby('Month')['Automobile_Sales'].mean().reset_index()

Y_chart2 = dcc.Graph(figure=px.line(mas,

x='Month',

y='Automobile_Sales',

title ='Yearly Automobile sales'))
```
  Plot bar chart for average number of vehicles sold during the given year
```python
avr_vdata=yearly_data.groupby('Year')['Automobile_Sales'].mean().reset_index()

Y_chart3 = dcc.Graph( figure=px.bar(avr_vdata,

x='Year',

y='Automobile_Sales',

title='Average Vehicles Sold by Vehicle Type in the year {}'.format(input_year)))
```
Total Advertisement Expenditure for each vehicle using pie chart
```python
exp_data=yearly_data.groupby('Vehicle_Type')['Advertising_Expenditure'].mean().reset_index()

Y_chart4 = dcc.Graph(figure=px.pie(exp_data,

values='Advertising_Expenditure',

names='Vehicle_Type',

title='Total Advertisement Expenditure for each vehicle'))
```
Returning the graphs for displaying Yearly data
```python
return [

html.Div(className='chart-item', children=[html.Div(children=Y_chart1),html.Div(children=Y_chart1)],style={'flex'}),

html.Div(className='chart-item', children=[html.Div(children=Y_chart2),html.Div(children=Y_chart2)],style={'flex'}),

html.Div(className='chart-item', children=[html.Div(children=Y_chart3),html.Div(children=Y_chart3)],style={'flex'}),

html.Div(className='chart-item', children=[html.Div(children=Y_chart4),html.Div(children=Y_chart4)],style={'flex'})

]

else:

return  None
```
  

### Run the Dash app
```python
if  __name__ == '__main__':

app.run_server(debug=True)
```

