import streamlit as st
import google.generativeai as genai
from dotenv import load_dotenv
import os

from tools import search_travel
from maps import google_maps_link

# -----------------------------
# Load Environment Variables
# -----------------------------
load_dotenv()

genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))

model = genai.GenerativeModel("gemini-2.5-flash")

# -----------------------------
# Streamlit Page
# -----------------------------
st.set_page_config(
    page_title="AI Travel Planner Agent",
    page_icon="✈️",
    layout="wide"
)

st.title("✈️ AI Travel Planner Agent")
st.write("Plan your perfect trip using AI + Live Web Search")

# -----------------------------
# User Inputs
# -----------------------------

col1, col2 = st.columns(2)

with col1:
    source = st.text_input("Source")
    destination = st.text_input("Destination")

    days = st.number_input(
        "Number of Days",
        min_value=1,
        max_value=30,
        value=3
    )

with col2:
    budget = st.number_input(
        "Budget (₹)",
        min_value=1000,
        value=10000
    )

    interest = st.selectbox(
        "Interest",
        [
            "Beach",
            "Adventure",
            "Food",
            "Nature",
            "Shopping",
            "Temple",
            "Wildlife"
        ]
    )

# -----------------------------
# Generate Plan
# -----------------------------

if st.button("Generate Travel Plan"):

    with st.spinner("🔍 Searching latest travel information..."):

        search_results = search_travel(
            f"""
            Best tourist attractions in {destination}
            Best hotels in {destination}
            Best restaurants in {destination}
            Best things to do in {destination}
            """
        )

    prompt = f"""
You are an expert AI Travel Planner.

Use the latest travel information below while creating the itinerary.

{search_results}

Travel Details

Source: {source}

Destination: {destination}

Days: {days}

Budget: ₹{budget}

Interest: {interest}

Generate a complete travel guide.

Include:

1. Trip Summary

2. Day-wise itinerary

3. Tourist attractions

4. Best restaurants

5. Best hotels

6. Local transport

7. Budget breakdown

8. Packing checklist

9. Travel tips

10. Safety advice

Return everything in beautiful markdown format.
"""

    with st.spinner("🤖 AI is planning your trip..."):
        response = model.generate_content(prompt)

    st.success("✅ Trip Generated Successfully!")

    st.markdown(response.text)

    # -----------------------------
    # Google Maps Links
    # -----------------------------

    st.subheader("📍 Google Maps")

    st.markdown(
        f"### 🗺️ Destination\n"
        f"[Open {destination} in Google Maps]({google_maps_link(destination)})"
    )

    st.markdown(
        f"### 🍴 Restaurants\n"
        f"[Best Restaurants in {destination}]({google_maps_link('Best restaurants ' + destination)})"
    )

    st.markdown(
        f"### 🏨 Hotels\n"
        f"[Best Hotels in {destination}]({google_maps_link('Best hotels ' + destination)})"
    )

    st.markdown(
        f"### 🎯 Tourist Attractions\n"
        f"[Top Attractions in {destination}]({google_maps_link('Tourist attractions ' + destination)})"
    )

    # -----------------------------
    # Download
    # -----------------------------

    st.download_button(
        label=" Download Itinerary",
        data=response.text,
        file_name="travel_plan.txt",
        mime="text/plain"
    )
