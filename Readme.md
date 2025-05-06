from dotenv import load_dotenv
import streamlit as st

st.set_page_config(page_title="A10_SUPER_BOT", layout="centered")
import os
import google.generativeai as genai

# Load environment variables
load_dotenv()
GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")

# Configure Gemini
genai.configure(api_key=GOOGLE_API_KEY)

# Default model list
MODELS = ["gemini-1.5-pro", "gemini-1.0-pro", "gemini-pro"]

# Initialize session state for chat history
if "messages" not in st.session_state:
    st.session_state.messages = []

# Sidebar with settings
with st.sidebar:
    st.title("⚙️ Settings")
    selected_model = st.selectbox("Model", MODELS, index=0)
    temperature = st.slider("Creativity (Temperature)", 0.0, 1.0, 0.7, 0.1)
    st.markdown("---")
    if st.button("🧹 Clear Chat History"):
        st.session_state.messages = []
        st.success("Chat history cleared!")

    st.markdown("### 📤 Export")
    if st.session_state.messages:
        history_text = "\n\n".join([f"**You:** {msg['user']}\n**Bot:** {msg['bot']}" for msg in st.session_state.messages])
        st.download_button("Download Chat (.txt)", history_text, file_name="chat_history.txt")

# Main app interface

st.title("🤖 A10 SUPER BOT")
st.markdown("Ask your question and get instant responses powered by Gemini!")

# User input
user_input = st.text_input("💬 Your Question:", key="input")

# Function to get Gemini response
def get_gemini_response(query, model_name, temperature):
    try:
        model = genai.GenerativeModel(model_name)
        response = model.generate_content(query, generation_config={"temperature": temperature})
        return response.text
    except Exception as e:
        return f"❌ Error: {e}"

# Submit button
if st.button("🚀 Ask Now"):
    if user_input.strip() == "":
        st.warning("Please enter a question.")
    else:
        with st.spinner("Thinking..."):
            answer = get_gemini_response(user_input, selected_model, temperature)
            st.session_state.messages.append({"user": user_input, "bot": answer})
            st.success("Done!")

# Display chat history
if st.session_state.messages:
    st.markdown("### 📜 Chat History")
    for msg in reversed(st.session_state.messages):
        st.markdown(f"**🧑 You:** {msg['user']}")
        st.markdown(f"**🤖 Bot:** {msg['bot']}")
        st.markdown("---")


