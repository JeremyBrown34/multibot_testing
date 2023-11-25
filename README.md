# multibot_testing 

# Setup Python notebook


    import streamlit as st
    import openai
    import pandas as pd
    import random
    import json
    
    openai.api_key = 'API KEY GOES HERE' 


# Function to generate a question
    def generate_question(question_type):
        # Define prompts for different types of questions
        basic_prompt = "Create a simple, common question that a new Uber driver might ask about their job."
        edge_case_prompt = "Create a complex or safety-related question that a new Uber driver might ask."
    
    prompt = basic_prompt if question_type == 'basic' else edge_case_prompt
    
    try:
        response = openai.Completion.create(
            model="text-davinci-003",  # Using text model for question generation
            prompt=prompt,
            max_tokens=60
        )
        
        question = response.choices[0].text.strip()
        return question
    except Exception as e:
        st.error(f"Error generating question: {e}")
        return "Error generating question."

# Function to handle conversation
def handle_conversation(question):
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4-1106-preview",
            messages=[
                {"role": "system", "content": "You are a helpful and knowledgeable FAQ bot for new Uber drivers."},
                {"role": "user", "content": question}
            ],
            max_tokens=150  # Set max tokens to limit response length
        )
        
        if response['choices'][0]['message']['role'] == "assistant":
            answer = response['choices'][0]['message']['content']
            # Truncate the response to approximately 100 words if necessary
            answer_words = answer.split()[:100]
            answer = ' '.join(answer_words)
        else:
            answer = 'No response detected in the API response.'

        return question, answer
    except Exception as e:
        st.error(f"Error in conversation handling: {e}")
        return question, "Error fetching response."

# Streamlit UI setup
    st.markdown("<h1 style='text-align: center; color: white;'>Jeremy's Multi-Bot Testing Example [GPT-4 Turbo] ☄️🤖📊</h1>", unsafe_allow_html=True)
    st.markdown("<h3 style='text-align: center; color: grey;'>Customer Support Simulation</h3>", unsafe_allow_html=True)

# Session state to store conversation
    if 'conversation_history' not in st.session_state:
    st.session_state['conversation_history'] = []

# Button to initiate conversation
    if st.button('Start Simulation'):
    st.session_state['conversation_history'] = []  # Reset the conversation history
    with st.spinner('Generating conversation...'):
        for _ in range(5):  # Loop for 10 questions
            question_type = 'basic' if random.random() < 0.75 else 'edge_case'
            question = generate_question(question_type)
            question, response = handle_conversation(question)
            st.session_state.conversation_history.append({"User Bot Question": question, "Chatbot Response": response})
            st.text(f"Customer: {question}")
            st.text(f"Support Bot: {response}")

# Display a download button for the conversation history
    if st.session_state['conversation_history']:
    df_conversation = pd.DataFrame(st.session_state['conversation_history'])
    csv = df_conversation.to_csv(index=False).encode('utf-8')
    st.download_button(
        label="Download conversation as Excel",
        data=csv,
        file_name='conversation_history.csv',
        mime='text/csv',
        key='download-csv'
    )
