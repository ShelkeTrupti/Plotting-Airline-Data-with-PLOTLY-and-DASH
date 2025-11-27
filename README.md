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

