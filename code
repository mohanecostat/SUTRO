import streamlit as st
import random
import time
import pickle
from datetime import datetime
st.set_page_config(page_title="SUTRO", page_icon="✈️")
st.title("✈️SUTRO ")
st.markdown("### Study Until Time Runs Out")

# --- INITIALIZE STATE ---
states = {'booked': False,'spun': False,'flying': False,'boarding': '','destination': '','duration': 0,'start_time': 0.0}
for k, v in states.items():
    if k not in st.session_state:
        st.session_state[k] = v

# --- STATION NUMBER CONFIGURATION ---
STATION_NUMBERS = {"Cochin": 0,"Calicut": 35,"Coimbatore": 50,"Chennai": 65,"Bangalore": 90,"Mysore": 105,"Mumbai": 120}
AIRPORTS = list(STATION_NUMBERS.keys())

# --- DURATION CALCULATION ENGINE ---
def calculate_flight_duration(source, target):
    num1 = STATION_NUMBERS[source]
    num2 = STATION_NUMBERS[target]
    
    diff = abs(num1 - num2)  
    
    if diff < 25:
        return int(diff * 2)     
    elif diff > 105:
        return int(diff / 2)     
    else:
        return int(diff)        

# --- BINARY FILE OPERATIONS ---
DB_FILE = "daily_streaks.dat"

def get_daily_streaks():
    try:
        f = open(DB_FILE, "rb")



        data = pickle.load(f)
        f.close()
        return data
    except (FileNotFoundError, EOFError):
        return {}

def log_study_time(mins):
    hrs = mins / 60.0
    today = str(datetime.now().date())
    data = get_daily_streaks()
    
    data[today] = data.get(today, 0.0) + hrs
    
    f = open(DB_FILE, "wb")
    pickle.dump(data, f)
    f.close()

# --- SECTION 1: BOOKING & FLIGHT LOGS ---
col1,col3,col2 = st.columns([24,1,9])
with col2:
    st.markdown("### 📊 Logs")
    all_data = get_daily_streaks()
    sorted_dates = sorted(all_data.keys(), reverse=True)[:3]
    for d in sorted_dates:
        st.write(f"📅 {d}: *{all_data[d]:.2f}h*")
with col1:
    st.markdown("### 🎟️ Book Your Ticket")
    st.session_state.boarding = st.selectbox("Boarding Airport:", AIRPORTS, disabled=st.session_state.booked)
    
    if not st.session_state.booked and st.button("Confirm Boarding Station ➡️", use_container_width=True):
        st.session_state.booked = True
        st.rerun()
with col3:
    st.empty()

# --- SECTION 2: SPIN WHEEL ---
if st.session_state.booked:
    st.markdown("---")
    st.markdown("### 🎡 Destination Roulette")
    options=[]
    for a in AIRPORTS:
        if a !=st.session_state.boarding:
            options.append(a)


    
    if not st.session_state.spun:
        if st.button("🎰 SPIN WHEEL FOR DESTINATION", use_container_width=True):
            spin_space = st.empty()
            for _ in range(24):
                spin_space.markdown(f"<h3 style='text-align:center; color:#FF4B4B;'>🎡 {random.choice(options)}</h3>", unsafe_allow_html=True)
                time.sleep(0.06)
            
            chosen_dest = random.choice(options)
            st.session_state.destination = chosen_dest
            
            st.session_state.duration = calculate_flight_duration(st.session_state.boarding, chosen_dest)
            
            st.session_state.spun = True
            st.rerun()
    else:
        st.success(f"🎯 Assigned Destination Terminal: *{st.session_state.destination}*")
        st.info(f"⏱️ Flight Duration calculated automatically: *{st.session_state.duration} minutes*")

# --- SECTION 3: PRE-TAKEOFF BRIEFING ---
if st.session_state.spun and not st.session_state.flying:
    st.markdown("---")
    st.warning(f"🧑✈️ Cabin Crew: Flight from {st.session_state.boarding} to {st.session_state.destination} ({st.session_state.duration} mins) is preparing for takeoff. Sit back and focus.")
    if st.button("🚀 Take Off", use_container_width=True):
        st.session_state.flying = True
        st.session_state.start_time = time.time()
        st.rerun()

# --- SECTION 4: IN-FLIGHT COUNTDOWN TIMER ---
if st.session_state.flying:
    st.markdown("---")
    st.markdown(f"### ✈️ En Route: {st.session_state.boarding} ➔ {st.session_state.destination}")
    
    elapsed = int(time.time() - st.session_state.start_time)



    remaining = (st.session_state.duration * 60) - elapsed
    
    if remaining <= 0:
        log_study_time(st.session_state.duration)
        st.balloons()
        st.success(f"🎉 Congratulations! You successfully completed your flight to {st.session_state.destination}!")
        if st.button("Reset Terminal 🏡", use_container_width=True):
            for k in ['booked', 'spun', 'flying']:
                st.session_state[k] = False
            st.rerun()
    else:
        st.markdown(f"<h1 style='text-align:center; color:#1E3A8A;'>{remaining//60:02d}:{remaining%60:02d}</h1>", unsafe_allow_html=True)
        pct = min(elapsed / (st.session_state.duration * 60),1.0)
        st.progress(pct)
        
        if pct < 0.5:
            st.info("🚀 Climbing to cruising altitude. Stay sharp!")
        elif pct < 0.8:
            st.warning("✨ Halfway to your destination! Don't give up.")
        else:
            st.success("🏁 Beginning final descent. Finish strong!")
        
        if st.button("🚨 EMERGENCY LANDING", use_container_width=True):
            if elapsed // 60 > 0:
                log_study_time(elapsed // 60)
            for k in ['booked', 'spun', 'flying']:
                st.session_state[k] = False
            st.rerun()
            
        time.sleep(1)
        st.rerun()
