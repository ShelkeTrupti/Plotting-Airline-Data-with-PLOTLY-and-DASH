# Read the airline data into pandas dataframe
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go

airline_data =  pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/airline_data.csv', 
                            encoding = "ISO-8859-1",
                            dtype={'Div1Airport': str, 'Div1TailNum': str, 
                                   'Div2Airport': str, 'Div2TailNum': str})
                                   
# Preview the first 5 lines of the loaded data 
airline_data.head(5)

# Shape of the data
airline_data.shape

# Now randomly sample 500 data points. Setting the random state to be 42 so that we get same result.
data = airline_data.sample(n=500, random_state=42)
data
data.shape
data.columns


# Let us use a scatter plot to represent departure time changes with respect to airport distance
fig = go.Figure(data=go.Scatter(x=data['Distance'], y=data['DepTime'], mode='markers', marker=dict(color='blue')))
fig.update_layout(title='Distance vs Departure Time', xaxis_title='Distance', yaxis_title='DepTime')
fig.show()

# Let us now use a line plot to extract average monthly arrival delay time and see how it changes over the year.
line_data = data.groupby('Month')['ArrDelay'].mean().reset_index()
line_data
line_plot = go.Figure(data=go.Scatter(x=line_data['Month'], y=line_data['ArrDelay'], mode='lines', marker=dict(color='Blue')))
line_plot.update_layout(title='Average monthly arrival delay time', xaxis_title='Month', yaxis_title='ArrDelay')
line_plot.show()

# Let us use a bar chart to extract number of flights that goes to a destination.
bar_data = data.groupby(['DestState'])['Flights'].sum().reset_index()
bar_data
fig = px.bar(bar_data, x="DestState", y="Flights", title='Total number of flights to the destination state split by reporting airline') 
fig.show()

# Let us represent the distribution of arrival delay using a histogram.
# First we need to set missing values to zero in ArrDelay column.
data['ArrDelay'] = data['ArrDelay'].fillna(0)
histogram = px.histogram(data, x=data['ArrDelay'])
histogram.show()

# Let use a bubble plot to represent number of flights as per reporting airline.
bub_data = data.groupby('Reporting_Airline')['Flights'].sum().reset_index()  
bub_data
fig = px.scatter(bub_data, x="Reporting_Airline", y="Flights", size="Flights", hover_name="Reporting_Airline", title='number of flights per reporting airline', size_max=40) 
fig.show()

# Let use pie chart to represent the proportion of distance group by month (month indicated by numbers)
fig = px.pie(data, values='Month', names='DistanceGroup', title='Distance group proportion by month')
fig.show()

# Let use sunurst chart to represent the hierarchical view in othe order of month and destination state holding value of number of flights
fig =px.sunburst(
    data,
    path = ['Month', 'DestStateName'],
    values='Flights',
    title ="Flight Distribution Hierarchy"
)
fig.show()

# Analyze flight delays in a Dashboard:
# Import required Libraries
import pandas as pd
import plotly.graph_objects as go
import plotly.express as px
import dash
import dash_html_components as html
import dash_core_components as dcc
from dash.dependencies import Input, Output


# Read the airline data into pandas dataframe
airline_data =  pd.read_csv('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DV0101EN-SkillsNetwork/Data%20Files/airline_data.csv')
encoding = "ISO-8859-1",

# Create a dash application
app = dash.Dash(__name__)

# Build dash app layout
app.layout = html.Div(children=[ html.H1('Airline Performance Dashboard', 
                                style={'textAlign': 'center', 'color': '#000000',
                                'font-size': 30}),
                                html.Div(["Input Year: ", dcc.Input(id='input-year', value='2010', 
                                type='number', style={'height':'35px', 'font-size': 25}),], 
                                style={'font-size': 30, 'color': '#000000'}),
                                html.Br(),
                                html.Br(), 
                                # Segment 1
                                html.Div([
                                        html.Div(dcc.Graph(id='carrier-plot')),
                                        html.Div(dcc.Graph(id='weather-plot'))
                                ], style={'display': 'flex'}),
                                # Segment 2
                                html.Div([
                                        html.Div(dcc.Graph(id='nas-plot')),
                                        html.Div(dcc.Graph(id='security-plot'))
                                ], style={'display': 'flex'}),
                                # Segment 3
                                html.Div(dcc.Graph(id='late-plot'), style={'width':'65%'})
                                ])

# Review and add supporting Function
def compute_info(airline_data, entered_year):
    # Select data
    df =  airline_data[airline_data['Year']==int(entered_year)]
    # Compute delay averages
    avg_car = df.groupby(['Month','Reporting_Airline'])['CarrierDelay'].mean().reset_index()
    avg_weather = df.groupby(['Month','Reporting_Airline'])['WeatherDelay'].mean().reset_index()
    avg_NAS = df.groupby(['Month','Reporting_Airline'])['NASDelay'].mean().reset_index()
    avg_sec = df.groupby(['Month','Reporting_Airline'])['SecurityDelay'].mean().reset_index()
    avg_late = df.groupby(['Month','Reporting_Airline'])['LateAircraftDelay'].mean().reset_index()
    return avg_car, avg_weather, avg_NAS, avg_sec, avg_late

#  Add the application callback function & Perform Computation

@app.callback( [
               Output(component_id='carrier-plot', component_property='figure'),
               Output(component_id='weather-plot', component_property='figure'),
               Output(component_id='nas-plot', component_property='figure'),
               Output(component_id='security-plot', component_property='figure'),
               Output(component_id='late-plot', component_property='figure')
               ],
               Input(component_id='input-year', component_property='value'))


def get_graph(entered_year):
    

    avg_car, avg_weather, avg_NAS, avg_sec, avg_late = compute_info(airline_data, entered_year)
            
    # Line plot for carrier delay
    carrier_fig = px.bar(avg_car, x='Month', y='CarrierDelay', color='Reporting_Airline', title='Average carrrier delay time by airline')
    
    # Line plot for weather delay
    weather_fig = px.line(avg_weather, x='Month', y='WeatherDelay', color='Reporting_Airline', title='Average weather delay time by airline')
   
    # Line plot for nas delay
    nas_fig = px.histogram(avg_NAS, x='Month', y='NASDelay', color='Reporting_Airline', title='Average NAS delay time by airline')
   
    # Line plot for security delay
    sec_fig = px.scatter(avg_sec, x='Month', y='SecurityDelay', color='Reporting_Airline', title='Average security delay time by airline')
   
    # Line plot for late aircraft delay
    late_fig = px.bar(avg_late, x='Month', y='LateAircraftDelay', color='Reporting_Airline', title='Average late aircraft delay time by airline')
            
    return[carrier_fig, weather_fig, nas_fig, sec_fig, late_fig]

# Run the app
if __name__ == '__main__':
    app.run()
