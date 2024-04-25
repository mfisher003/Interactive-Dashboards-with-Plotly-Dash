## Creating interactive dashboards with Python | Plotly | Dash | IBM Cloud IDE

### Project Goal
Analyze historic tends in automobile sales during recession periods. 
This was the final assignment in module 7 associated with IBM Data Analyst Professional certificate.

### Python Commands Explored
- Create Dropdown menus
- HTML layout
- Callback functions
- Creating and displaying 4 interactive plots

### Here are a few screenshots of the interactive plots
![Yearly auto sales 2022](https://i.imgur.com/htXpSSY.png)

![Auto Sales during a recession 2022](https://i.imgur.com/7XMUYi3.png)

### Importing Python Libraries
```python
#!/usr/bin/env python

# coding: utf-8

import dash
from dash import dcc, html
from dash.dependencies import Input, Output
import pandas as pd
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
dropdown_options = [
    {'label': 'Yearly Statistics', 'value': 'Yearly Statistics'},
    {'label': 'Recession Period Statistics', 'value': 'Recession Period Statistics'}
]

app.layout = html.Div([
    html.H1("Automobile Statistics Dashboard",
            style={'textAlign': 'center', 'color': '#503D36', 'font-size': 24}),
    html.Div([
        html.Label("Selected Statistics:"),
        dcc.Dropdown(
            id='dropdown-statistics',
            options=dropdown_options
        )
    ]),
    html.Div(dcc.Dropdown(
        id='select-year',
        options=[{'label': i, 'value': i} for i in year_list],
        value='Select-year'
    )),
    html.Div([
        html.Div(id='output-container', className='chart-grid', style={'display': 'flex'})
    ])
])
```
### Creating Callbacks
Define the callback function to update the input container based on the selected statistics
```python
@app.callback(
    Output(component_id='select-year', component_property='disabled'),
    [Input(component_id='dropdown-statistics', component_property='value')]
)
def update_input_container(selected_statistics):
    if selected_statistics == 'Yearly Statistics':
        return False
    else:
        return True
```
  
### Callback for plotting
Define the callback function to update the input container based on the selected statistics
```python
@app.callback(
    Output(component_id='output-container', component_property='children'),
    [Input(component_id='dropdown-statistics', component_property='value'),
     Input(component_id='select-year', component_property='value')]
)
def update_output_container(selected_statistics, input_year):
    if selected_statistics == 'Recession Period Statistics':
        recession_data = data[data['Recession'] == 1]
        yearly_rec = recession_data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        R_chart1 = dcc.Graph(
            figure=px.line(
                yearly_rec,
                x='Year',
                y='Automobile_Sales',
                title="Average Automobile Sales fluctuation over Recession Period"
            )
        )

        average_sales = recession_data.groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        R_chart2 = dcc.Graph(
            figure=px.line(
                average_sales,
                x='Vehicle_Type',
                y='Automobile_Sales',
                title="Average number of vehicles sold by vehicle type during recessions"
            )
        )

        exp_rec = recession_data.groupby('Vehicle_Type')['Advertising_Expenditure'].mean().reset_index()
        R_chart3 = dcc.Graph(
            figure=px.pie(
                exp_rec,
                values='Advertising_Expenditure',
                names='Vehicle_Type',
                title="Total expenditure share by vehicle type during recessions"
            )
        )

        unemp_data = recession_data.groupby(['Vehicle_Type', 'unemployment_rate'])['Automobile_Sales'].mean().reset_index()
        R_chart4 = dcc.Graph(
            figure=px.bar(
                unemp_data,
                x='unemployment_rate',
                y='Automobile_Sales',
                color='Vehicle_Type',
                labels={'unemployment_rate': 'Unemployment Rate', 'Automobile_Sales': 'Average Automobile Sales'},
                title="Effect of Unemployment Rate on Sales of various Vehicle Types"
            )
        )

        return [
            html.Div(className='chart-row', children=[
                html.Div(className='chart-item', children=[html.Div(children=R_chart1)]),
                html.Div(className='chart-item', children=[html.Div(children=R_chart2)])
            ]),
            html.Div(className='chart-row', children=[
                html.Div(className='chart-item', children=[html.Div(children=R_chart3)]),
                html.Div(className='chart-item', children=[html.Div(children=R_chart4)])
            ])
        ]

    elif input_year and selected_statistics == 'Yearly Statistics':
        yas = data.groupby('Year')['Automobile_Sales'].mean().reset_index()
        Y_chart1 = dcc.Graph(
            figure=px.line(
                yas,
                x='Year',
                y='Automobile_Sales',
                title='Yearly Automobile sales'
            )
        )

        Y_chart2 = dcc.Graph(
            figure=px.line(
                data[data['Year'] == input_year],
                x='Month',
                y='Automobile_Sales',
                title='Total Monthly Automobile sales'
            )
        )

        avr_vdata = data[data['Year'] == input_year].groupby('Vehicle_Type')['Automobile_Sales'].mean().reset_index()
        Y_chart3 = dcc.Graph(
            figure=px.bar(
                avr_vdata,
                x='Vehicle_Type',
                y='Automobile_Sales',
                title='Average Vehicles Sold by Vehicle Type in the year'
            )
        )

        exp_data = data[data['Year'] == input_year].groupby('Vehicle_Type')['Advertising_Expenditure'].mean().reset_index()
        Y_chart4 = dcc.Graph(
            figure=px.pie(
                exp_data,
                values='Advertising_Expenditure',
                names='Vehicle_Type',
                title='Total Advertisement Expenditure for each vehicle'
            )
        )

        return [
            html.Div(className='chart-row', children=[
                html.Div(className='chart-item', children=[html.Div(children=Y_chart1)]),
                html.Div(className='chart-item', children=[html.Div(children=Y_chart2)])
            ]),
            html.Div(className='chart-row', children=[
                html.Div(className='chart-item', children=[html.Div(children=Y_chart3)]),
                html.Div(className='chart-item', children=[html.Div(children=Y_chart4)])
            ])
        ]

    else:
        return None
```
  
### Run the Dash app
```python
if  __name__ == '__main__':

app.run_server(debug=True)
```

