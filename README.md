# FinalProject

import pandas as pd
import matplotlib.pyplot as plt
import streamlit as st
import numpy as np
import pydeck as pdk


def read_data():
    df = pd.read_csv('Skyscrapers2021.csv').set_index("RANK")
    return df


# filter the data
def filter_data(sel_cities, max_floors, min_completion):
    df = read_data()
    df.dropna(subset=["FLOORS"], inplace=True)
    df = df.loc[df['CITY'].isin(sel_cities)]
    df = df.loc[df['FLOORS'] < max_floors]
    df = df.loc[df['COMPLETION'] > min_completion]

    return df


# determine all cities available
def all_cities():
    df = read_data()
    lst = []
    for ind, row in df.iterrows():
        if row['CITY'] not in lst:
            lst.append(row['CITY'])

    return lst


# count all cities available
def count_cities(cities, df):
    df = read_data()
    lst = [df.loc[df['CITY'].isin([CITY])].shape[0] for CITY in cities]

    return lst


# determine number of city floors
def city_floors(df):
    floors = [row['FLOORS'] for ind, row in df.iterrows()]
    cities = [row['CITY'] for ind, row in df.iterrows()]
    dict = {}
    for city in cities:
        dict[city] = []

    for i in range(len(floors)):
        dict[cities[i]].append(floors[i])

    return dict


#   define optional colors
def choose_color():
    lst = ["red","blue","green","gray", "magenta", "yellow"]

    return lst


#   counts skyscrapers per city
def skyscrapers_per_city(df):
    cities = [row['CITY'] for ind, row in df.iterrows()]

    city_freq = {}
    for city in cities:
        city_freq[city] = 0

    for key in city_freq.keys():
        count = 0
        for city in cities:
            if city == key:
                count += 1
        city_freq[key] = count

    return city_freq


# finds average floors per skyscraper in selected city
def city_floor_avg(dict_floors):
    dict = {}
    for key in dict_floors.keys():
        dict[key] = np.mean(dict_floors[key])

    return dict


# displays the percentage breakdown of skyscrapers in selected cities
def generate_pie_chart(counts, sel_cities):
    plt.figure()

    explodes = [0 for i in range(len(counts))]
    maximum = counts.index(np.max(counts))
    explodes[maximum] = 0.25

    plt.pie(counts, labels=sel_cities, explode=explodes, autopct="%.2f")
    plt.title(f"Cities with Tallest Buildings: {', '.join(sel_cities)}")
    return plt


# displays the number breakdown of skyscrapers in selected cities
def generate_bar_chart(dict_averages, color):
    plt.figure()

    x = dict_averages.keys()
    y = dict_averages.values()
    plt.bar(x, y, color=color)
    plt.xticks(rotation=45)
    plt.ylabel("Number of Skyscrapers")
    plt.xlabel("Cities")
    plt.title(f"Number of Skyscrapers in Cities: {', '.join(dict_averages.keys())}")

    return plt


# displays the average floor numbers for skyscrapers in selected city
def generate_bar_chart2(dict_averages, color):
    plt.figure()

    x = dict_averages.keys()
    y = dict_averages.values()
    plt.bar(x, y, color=color)
    plt.xticks(rotation=45)
    plt.ylabel("Floors")
    plt.xlabel("Cities")
    plt.title(f"Average Floor Number for Skyscrapers in Cities: {', '.join(dict_averages.keys())}")

    return plt


# displays a map of selected cities and skyscrapers within them
def generate_map(df):
    map_df = df.filter(['NAME','Latitude','Longitude'])
    viewState = pdk.ViewState(latitude=map_df["Latitude"].mean(),
                              longitude=map_df["Longitude"].mean(),
                              zoom=8)
    layer = pdk.Layer('ScatterplotLayer', data=map_df,
                      get_position='[Longitude, Latitude]',
                      get_radius=800,
                      get_color = [200, 175, 950],
                      pickable=True)
    tool_tip = {'html': 'Listing:<br>/ <b>{NAME}</b>',
                'style': {'backgroundColor': 'steelblue', 'color': 'white'}}
    map = pdk.Deck(map_style='mapbox://styles/mapbox/light-v9',
                   initial_view_state=viewState,
                   layers=[layer],
                   tooltip=tool_tip)

    st.pydeck_chart(map)


# extra stats, so the user knows some background info about all the cities
def statistics():
    df = read_data()
    st.write('--------------')
    st.title("Statistics:")
    st.write("""
     """)
    total_num = df['NAME'].count()
    st.write(f"The **total** buildings in the data set are: *{str(total_num)}*")

    st.write(
        "Below is a Pivot Table showing total number of buildings in the city along with max and min heights in that city.")
    pivot = df.pivot_table(index=['CITY'], values=['Feet'], aggfunc={'max', 'min', 'count'})
    st.write(pivot)


# defining the main function
def main():
    st.title("Data Visualization with Python")
    st.write("Welcome to this Skyscraper 2021 Data! On this page, you can learn about the tallest skyscrapers around the world."
             " You can chose skyscrapers based on the maximum number of floors and completion date."
             " Don't forget to choose your favorite color for the charts! "
             "Hopefully, you can decide the next destination for your trip.")
    st.write("Open the sidebar for options.")
    st.write("__Please enter your name to get started.__")

    st.sidebar.write("Please choose your options to display data.")
    cities = st.sidebar.multiselect("Select cities: ", all_cities())

    colors = st.sidebar.multiselect("Select your favorite color: ", choose_color())
    colors2 = st.sidebar.multiselect("Select your second favorite color: ", choose_color())

    max_floor = st.sidebar.slider("Max Floor: ", 50, 200)
    min_complete = st.sidebar.slider("Min Completion: ", 1920, 2020)

    name_cols = st.columns(3)
    first_name = name_cols[0].text_input("First Name")
    middle_initial = name_cols[1].text_input("Middle Initial")
    last_name = name_cols[2].text_input("Last Name")

    button1 = st.sidebar.button("Locate Skyscrapers")

    data = filter_data(cities, max_floor, min_complete)
    series = count_cities(cities, data)

    if len(cities) > 0 and max_floor > 0 and min_complete > 0:
        if button1:
            st.write("View a map of buildings")
            generate_map(data)

        st.write("View a pie chart")
        st.pyplot(generate_pie_chart(series, cities))

        st.write("View a bar chart")
        st.pyplot(generate_bar_chart(city_floor_avg(skyscrapers_per_city(data)), colors))

        st.write("View a bar chart")
        st.pyplot(generate_bar_chart2(city_floor_avg(city_floors(data)), colors2))

    winning_city = st.radio("Select what city you want to visit: ", cities)

    statistics()

    st.write("  ")
    st.write("  ")
    st.write("  ")
    st.write("  ")

    st.write("Enjoy your trip to ", winning_city, first_name, middle_initial, ".", last_name, "!")


main()
