import speech_recognition as sr
import pyttsx3
import webbrowser
import datetime
import os

# Replace with your actual OpenAI API key
OPENAI_API_KEY = "sk-..."  

# Initialize text-to-speech engine
engine = pyttsx3.init()

# Speak text aloud
def speak(text):
    engine.say(text)
    engine.runAndWait()

# Listen for user input (voice)
def listen():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        audio = r.listen(source)
    try:
        command = r.recognize_google(audio)
        print("You said:", command)
        return command.lower()
    except Exception:
        speak("Sorry, I could not understand.")
        return ""

# Use OpenAI to answer questions
def gpt_answer(prompt):
    import openai
    openai.api_key = OPENAI_API_KEY
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo", 
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message['content'].strip()

# Main assistant logic
def assistant():
    speak("Hello! How can I assist you today?")
    while True:
        command = listen()
        if not command:
            continue

        # Stop assistant
        if "exit" in command or "quit" in command or "stop" in command:
            speak("Goodbye!")
            break

        # Open a website
        elif "open" in command and "website" in command:
            speak("Which website should I open?")
            site = listen()
            url = f"https://{site}"
            webbrowser.open(url)
            speak(f"Opening {site}")
        
        # Tell the time
        elif "time" in command:
            now = datetime.datetime.now().strftime("%I:%M %p")
            speak(f"The time is {now}")

        # Set a reminder (simple file-based)
        elif "remind me" in command:
            speak("What should I remind you about?")
            reminder = listen()
            with open("reminders.txt", "a") as f:
                f.write(reminder + "\n")
            speak("Reminder noted.")

        # Read reminders
        elif "read reminders" in command:
            if os.path.exists("reminders.txt"):
                with open("reminders.txt", "r") as f:
                    reminders = f.read()
                speak("Your reminders are: " + reminders)
            else:
                speak("You have no reminders.")

        # General question (using GPT)
        else:
            answer = gpt_answer(command)
            speak(answer)

if __name__ == "__main__":
    assistant()
    
