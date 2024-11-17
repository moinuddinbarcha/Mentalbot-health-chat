# Mentalbot-health-chat<break>
    import json
import random
import gradio as gr
from difflib import SequenceMatcher  # For fuzzy matching

# Load intents from the JSON file
with open("intents.json", "r", encoding="utf-8") as file:
    data = json.load(file)
    intents = data["intents"]

# Prepare patterns and responses for matching
patterns_responses = []
for intent in intents:
    if "patterns" in intent and "responses" in intent:  # Ensure both keys exist
        for pattern in intent["patterns"]:
            patterns_responses.append((pattern.lower(), intent["responses"]))
    else:
        print(f"Skipping intent without 'patterns' or 'responses': {intent}")

# Function to find the best match using fuzzy matching
def find_best_response(user_input):
    user_input = user_input.lower()
    best_match = None
    best_ratio = 0.0

    # Iterate through all patterns and calculate similarity
    for pattern, responses in patterns_responses:
        similarity = SequenceMatcher(None, user_input, pattern).ratio()
        if similarity > best_ratio:
            best_ratio = similarity
            best_match = responses 

    # Return a response if a good match is found
    if best_ratio > 0.6:  # Adjusted threshold for better accuracy
        return random.choice(best_match)

    # Fallback responses
    fallback_responses = [
        "I'm here to listen. Could you tell me more?",
        "Can you clarify that a bit for me?",
        "I'm not sure I understand. Could you explain further?",
        "Please share more details so I can help better."
    ]
    return random.choice(fallback_responses)

# Chatbot function to interact with the user
def mental_health_chatbot(user_input):
    return find_best_response(user_input)

# Create the Gradio Interface
interface = gr.Interface(
    fn=mental_health_chatbot,
    inputs=gr.Textbox(label="How may I assist you?", placeholder="Type your thoughts here..."),
    outputs=gr.Textbox(label="Pandora (Your AI Companion)"),
    title="Pandora - Your Mental Health Chatbot",
    description="I'm here to talk about how you're feeling and provide support. Share your thoughts with me.",
    theme="compact",
    allow_flagging="never",
)

# Launch the chatbot
if __name__ == "__main__":
    interface.launch()
