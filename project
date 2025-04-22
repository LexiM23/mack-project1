import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import pydeck as pdk

st.set_page_config(page_title="Beginner Geologists Guide to Volcanoes", layout="wide")


@st.cache_data
def load_data():
    try:
        df = pd.read_csv("volcanoes.csv", skiprows=1, encoding="latin1")
    except FileNotFoundError:
        st.error("CSV file not found. Please check the filename and try again.")
        return pd.DataFrame()

    df.columns = [
        "Volcano_Number", "Name", "Country", "Region", "Subregion", "Landform",
        "Primary_Type", "Activity_Evidence", "Last_Eruption", "Lat", "Lon",
        "Elevation_m", "Tectonic_Setting", "Dominant_Rock_Type"
    ]

    df.dropna(subset=['Lat', 'Lon'], inplace=True)
    df['Elevation_m'] = pd.to_numeric(df['Elevation_m'], errors='coerce')
    df['Elevation_km'] = df['Elevation_m'] / 1000

    # [PY4] List comprehension to extract year from eruption string
    eruption_years = [
        int(e[:4]) if isinstance(e, str) and e[:4].isdigit() else None
        for e in df['Last_Eruption']
    ]
    df['Eruption_Year'] = eruption_years
    df['Country'] = df['Country'].str.strip()

    return df


# [PY1] Function with a default parameter
def get_eruptions_by_year_range(start=2000, end=2025):
    return df[(df['Eruption_Year'] >= start) & (df['Eruption_Year'] <= end)]


# [PY2] Function returning two values
def get_eruption_range():
    return df['Eruption_Year'].min(), df['Eruption_Year'].max()


df = load_data()

if not df.empty:
    st.title("Beginner Geologists Guide to Volcanoes")

    st.markdown("""
    ## Volcanic Forecasting

    This app explores over 1,500 volcanoes using data provided by the Smithsonian.
    Compare countries, locate hot spots, and see how eruptions have increased over time. 
    This beginner’s guide to volcanoes will prepare any aspiring geologist!
    """)

    # [PY1] Using the default call
    recent_eruptions_df = get_eruptions_by_year_range()

    # [PY1] Call again with user-defined range
    st.title("Eruption Frequency by Year Range")
    st.write("This chart shows the number of eruptions within your selected year range. "
             "Geologists can observe decade-to-decade patterns to predict future eruptions.")
    min_year, max_year = st.slider("Select Eruption Year Range", 1800, 2025, (1900, 2025))
    filtered_df = get_eruptions_by_year_range(min_year, max_year)

    # [PY2] Use returned values to display year range in dataset
    min_val, max_val = get_eruption_range()
    st.write(f"Full eruption year range in the dataset: {min_val} to {max_val}")

    fig1, ax1 = plt.subplots(figsize=(10, 6))
    ax1.hist(filtered_df['Eruption_Year'].dropna(), bins=25, color='orangered', edgecolor='black')
    ax1.set_title(f"Eruption Frequency: {min_year}–{max_year}")
    ax1.set_xlabel("Year")
    ax1.set_ylabel("Number of Volcanoes")
    st.pyplot(fig1)

    # Country Activity Visualization
    st.title("Volcanic Activity by Country")
    st.write(
        "This chart shows how many volcanoes fall into each activity category (e.g., historical, dormant) for a selected country.")
    country = st.selectbox("Select a Country to Analyze", sorted(df['Country'].dropna().unique()))
    country_data = df[df['Country'] == country]
    status_counts = country_data['Activity_Evidence'].value_counts()

    # [PY5] Accessing dictionary items
    st.subheader(f"Volcano Status in {country}")
    for status, count in status_counts.to_dict().items():
        st.write(f"{status}: {count} volcanoes")

    fig2, ax2 = plt.subplots()
    ax2.bar(status_counts.index, status_counts.values, color='firebrick')
    ax2.set_title(f"Volcano Activity Evidence in {country}")
    ax2.set_ylabel("Number of Volcanoes")
    plt.xticks(rotation=30, ha='right', fontsize=10)
    st.pyplot(fig2)

    # Map of Recent Eruptions
    st.title("Volcanoes That Have Erupted in the Last Decade")
    st.write("This map highlights volcanoes that erupted between 2015 and 2025. "
             "It helps geologists predict active zones and identify regions to monitor for safety.")
    recent_only_df = get_eruptions_by_year_range(2015, 2025)

    layer = pdk.Layer(
        "ScatterplotLayer",
        data=recent_only_df,
        get_position='[Lon, Lat]',
        get_color='[255, 69, 0, 180]',
        get_radius=40000,
        pickable=True,
    )

    view_state = pdk.ViewState(latitude=10, longitude=0, zoom=1)

    deck = pdk.Deck(
        layers=[layer],
        initial_view_state=view_state,
        tooltip={"text": "{Name}\nCountry: {Country}\nLast Eruption: {Last_Eruption}"}
    )

    st.pydeck_chart(deck)

    # Historical Region Table
    st.title("Historical Eruption Frequencies by Region")
    st.write("This table counts how many eruptions happened across different historical periods in each region. "
             "Patterns help geologists understand long-term volcanic activity across the globe.")
    region_years = df.dropna(subset=['Region', 'Eruption_Year'])
    region_grouped = region_years.groupby([
        'Region',
        pd.cut(region_years['Eruption_Year'], bins=[0, 1800, 1900, 1950, 2000, 2025])
    ]).size().unstack(fill_value=0)

    st.dataframe(region_grouped)
