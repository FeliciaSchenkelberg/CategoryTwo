## Algorithms and Data Structure

Category Two is concerned with Algorithms and Data Structure. I will illustrate my work, using an artifact from a previous course. The artifact I will be using for my first category will be from CS 340: Advanced Programming Concepts. In this course, I learned how to apply the concepts and principles of database systems to the development of client/server applications that link client code to databases.

I developed successful CRUD routines in Python for MongoDB and then created a fully functional MongoDB dashboard. This process is illistrated below:

### CRUD routines for MongoDB

```markdown
from pymongo import MongoClient
from bson.objectid import ObjectId

class AnimalShelter(object):
    """ CRUD operations for Animal collection in MongoDB """
        
    def __init__(self, username, password):
        # Initializing the MongoClient. This helps to 
        # access the MongoDB databases and collections. 
        self.client = MongoClient('mongodb://%s:%s@localhost:50358/?authMechanism=DEFAULT&authSource=AAC' % (username, password))
        self.database = self.client['AAC']

    def create(self, data):
        if data is not None:
            self.database.animals.insert(data)
            return True
        else:
            print('Nothing to save, because data parameter is empty')
            return False
            
    def read(self, data):
        if data is not None:
            return self.database.animals.find(data)
        else:
            print('Nothing to read, because data parameter is empty')
            return False

    def update(self, data, change):
        if data is not None:
            return self.database.animals.update(data,{ "$set": change})
        else:
            print('Nothing to update, because data parameter is empty')
            return False

    def delete(self, data):
        if data is not None:
            return self.database.animals.delete_one(data)
        else:
            print('Nothing to delete, because data parameter is empty')
            return False
```


### The MongoDB Dashboard

Tthe dashboard filter options should be able to properly retrieve data from the database and develop the controller pieces to create interactive options that allow for the selection of data based on the filtering functions. Ensuring that the dashboard widgets receive input from the interactive options and present those dynamic updates to the client.

```markdown
from jupyter_plotly_dash import JupyterDash

import dash
import dash_leaflet as dl
import dash_core_components as dcc
import dash_html_components as html
import plotly.express as px
import dash_table as dt
from dash.dependencies import Input, Output, State

import os
import numpy as np
import pandas as pd
from pymongo import MongoClient
from bson.json_util import dumps

from CRUD import AnimalShelter

###########################
# Data Manipulation / Model
###########################
# For username, password, and CRUD Python module name
username = "aacuser"
password = "Hello"
shelter = AnimalShelter(username, password)

# Class read method must support return of cursor object 
df = pd.DataFrame.from_records(shelter.read({}))

#########################
# Dashboard Layout / View
#########################
app = JupyterDash('Felicia Schenkelberg')

# Addition of Grazioso Salvare’s logo
image_filename = 'GraziosoSalvareLogo.png'
encoded_image = base64.b64encode(open(image_filename, 'rb').read())

app.layout = html.Div([
    html.Img(id='customer-image',src='data:image/png;base64,{}'.format(encoded_image.decode()),alt='customer image'),
    html.Center(html.B(html.H1('SNHU CS-340 Dashboard'))),
    html.Hr(),
    html.Div(
        
# Code for the interactive filtering options.
        className='row',
             style={'display' : 'flex'},
                  children=[
                          html.Button(id='submit-button-one', n_clicks=0, children='Water Rescue'),
                          html.Button(id='submit-button-two', n_clicks=0, children='Mountain or Wilderness Rescue'),
                          html.Button(id='submit-button-three', n_clicks=0, children='Disaster or Individual Tracking'),
                          html.Button(id='submit-button-four', n_clicks=0, children='Reset')
                  ]),
    dt.DataTable(
        id='datatable-interactivity',
        columns=[
            {"name": i, "id": i, "deletable": False, "selectable": True} for i in df.columns
        ],
        data=df.to_dict('records'),
# Setting up the features for the interactive data table to make it user-friendly for my client
        editable=False,
        sort_action="native",
        sort_mode="multi",
        column_selectable=False,
        row_selectable=False,
        row_deletable=False,
        selected_columns=[],
        selected_rows=[],
        page_action="native",
        page_current= 0,
        page_size= 10,
    ),
    html.Br(),
    html.Hr(),
# Setting up the dashboard so that my chart and geolocation chart are side-by-side
    html.Div(className='row',
         style={'display' : 'flex'},
             children=[
        html.Div(
            id='graph-id',
            className='col s12 m6',

            ),
        html.Div(
            id='map-id',
            className='col s12 m6',
            )
        ])
])

#############################################
# Interaction Between Components / Controller
############################################# 

### Filter interactive data table with MongoDB queries
@app.callback(Output('datatable-interactivity',"data"), 
              [Input('submit-button-one', 'n_clicks'),Input('submit-button-two', 'n_clicks'),
               Input('submit-button-three', 'n_clicks'),Input('submit-button-four', 'n_clicks'),
              ])

def on_click(bt1,bt2,bt3,bt4):
    # start case
    if (int(bt1) > 1):
        df = pd.DataFrame.from_records(shelter.read({'$and': [{'$or': [
        {'breed':'Labrador Retriever Mix'}, {'breed':'Chesapeake Bay Retriever'}, {'breed':'Newfoundland'}]},
        {'sex_upon_outcome':'Intact Female'}, {'age_upon_outcome_in_weeks':{'$lte':26, 'gte':156}}]}))
    elif (int(bt2) > 1):
        df = pd.DataFrame(list(shelter.read({'$and': [{'$or': [
        {'breed':'Labrador Retriever Mix'}, {'breed':'Chesapeake Bay Retriever'}, {'breed':'Newfoundland'}]},
        {'sex_upon_outcome':'Intact Female'}, {'age_upon_outcome_in_weeks':{'$lte':26, 'gte':156}}]})))
    elif (int(bt3) > 1):
        df = pd.DataFrame(list(shelter.read({'$and': [{'$or': [
        {'breed':'German Shepherd'}, {'breed':'Alaskan Malamute'}, {'breed':'Old English Sheepdog'},
        {'breed':'Siberian Husky'}, {'breed':'Rottweiler'}]}, {'sex_upon_outcome':'Intact Male'},
        {'age_upon_outcome_in_weeks':{'$lte':20, 'gte':300}}]})))
    elif (int(bt4) > 1):
        df = pd.DataFrame(list(shelter.read()))
    
        
    return df.to_dict('records')

 
    columns=[{"name": i, "id": i, "deletable": False, "selectable": True} for i in df.columns]
    data=df.to_dict('records')
                
    return (data,columns)

@app.callback(
    Output('datatable-id', 'style_data_conditional'),
    [Input('datatable-id', 'selected_columns')]
)
def update_styles(selected_columns):
    return [{
        'if': { 'column_id': i },
        'background_color': '#D2F3FF'
    } for i in selected_columns]

@app.callback(
    Output('graph-id', "children"),
    [Input('datatable-id', "derived_viewport_data")])
def update_graphs(viewData):
    df = pd.DataFrame.from_dict(viewData)
    return [
        dcc.Graph(            
            figure = px.pie(df, values=values, names=names, title='Percentage of breeds available')
        )    
    ]
                          
@app.callback(
    Output('map-id', "children"),
    [Input('datatable-id', "derived_viewport_data")])
                          
def update_map(viewData):
# The code for the geolocation chart
    viewDF = pd.DataFrame.from_dict(viewData)
    dff = viewDF.loc[rows]
    # Austin TX is at [30.75,-97.48]
    return [
        dl.Map(style={'width': '1000px', 'height': '500px'}, center=[30.75,-97.48], zoom=15, children=[
            dl.TileLayer(id="base-layer-id"),
            # Marker with tool tip and popup
            dl.Marker(position=[dff.loc[0,'location_lat'],dff.loc[0,'location_long']], children=[
                dl.Tooltip(dff['breed']),
                dl.Popup([
                    html.H1("Animal Name"),
                    html.P(dff.loc[0,'name'])
                ])
            ])
        ])
    ]


app
```

### Support or Contact

In completing this artifact, I gained a better understanding of the principles and best practices used to develop high-quality software. For more details, you can view this artifact [Here](http://localhost:8888/notebooks/TestDashboardOne.ipynb) or download this artifact [Here](https://github.com/FeliciaSchenkelberg/CS340-T5460).

[Back to Welcome Page](https://feliciaschenkelberg.github.io/).
