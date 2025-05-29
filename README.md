import tkinter as tk
from tkinter import scrolledtext
from threading import Thread
import pyttsx3
import speech_recognition as sr
import datetime
import webbrowser
import random
from plyer import notification
import pyautogui
import wikipedia
import pywhatkit as pwk
import gemini_request as ai

#Text-to-Speech 
engine = pyttsx3.init()
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[1].id)
engine.setProperty("rate", 170)

def speak(audio):
    engine.say(audio)
    engine.runAndWait()

# Themes
themes = {
    "dark": {
        "bg": "#222222", "fg": "white",
        "button_bg": "#1db954", "text_bg": "#333333", "text_fg": "white"
    },
    "light": {
        "bg": "#f5f5f5", "fg": "#222222",
        "button_bg": "#3a1db9", "text_bg": "#ffffff", "text_fg": "#222222"
    }
}
current_theme = "dark"

#GUI Functions
def update_output(text):
    output_box.insert(tk.END, text + "\n")
    output_box.see(tk.END)

def toggle_theme():
    global current_theme
    current_theme = "light" if current_theme == "dark" else "dark"
    apply_theme()

def apply_theme():
    theme = themes[current_theme]
    app.config(bg=theme["bg"])
    label.config(bg=theme["bg"], fg=theme["fg"])
    output_box.config(bg=theme["text_bg"], fg=theme["text_fg"], insertbackground=theme["text_fg"])
    listen_button.config(bg=theme["button_bg"], fg="white", activebackground=theme["button_bg"])
    toggle_button.config(bg=theme["button_bg"], fg="white", activebackground=theme["button_bg"])

# Speech Recognition
def command():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        update_output("🎙️ Listening...")
        r.adjust_for_ambient_noise(source)
        audio = r.listen(source)
    try:
        content = r.recognize_google(audio, language='en-in')
        update_output(f"🗣️ You said: {content}")
        return content.lower()
    except Exception:
        update_output("❌ Error recognizing voice. Please try again.")
        return ""

def continuous_listen():
    listen_button.config(state=tk.DISABLED)
    while True:
        request = command()
        if request:
            process_command(request)

#Command Processor
def process_command(request):
    if "Jarvis" in request:
        speak("Welcome, how can I help you?")
        update_output("Jarvis: Welcome, how can I help you?")
    elif "play music" in request:
        speak("Playing music")
        songs = [
            "https://www.youtube.com/watch?v=vEe-UgJvUHE",
            "https://www.youtube.com/watch?v=mk3XycambgI&list=RDMM",
            "https://www.youtube.com/watch?v=2Vv-BfVoq4g&list=RDMM",
            "https://www.youtube.com/watch?v=OTMQJ656r-M&list=RDMM",
            "https://www.youtube.com/watch?v=TIQ_Gh7clQ8&list=RDMM"
        ]
        webbrowser.open(random.choice(songs))
    elif "say time" in request:
        now_time = datetime.datetime.now().strftime("%H:%M")
        speak(f"Now time is {now_time}")
        update_output(f"Jarvis: Now time is {now_time}")
    elif "say date" in request:
        now_date = datetime.datetime.now().strftime("%d:%m")
        speak(f"Now date is {now_date}")
        update_output(f"Jarvis: Now date is {now_date}")
    elif "new task" in request:
        task = request.replace("new task", "").strip()
        if task:
            with open("todo.txt", "a") as file:
                file.write(task + "\n")
            speak(f"Task added: {task}")
            update_output(f"Jarvis: Task added - {task}")
    elif "speak task" in request:
        with open("todo.txt", "r") as file:
            tasks = file.read()
        speak("Today's tasks are: " + tasks)
        update_output("Jarvis: " + tasks)
    elif "show work" in request:
        with open("todo.txt", "r") as file:
            tasks = file.read()
        notification.notify(title="Today's work", message=tasks)
        update_output("Jarvis: Showing work in notification")
    elif "open youtube" in request:
        webbrowser.open("https://www.youtube.com")
    elif "open facebook" in request:
        webbrowser.open("https://www.facebook.com")
    elif "open instagram" in request:
        webbrowser.open("https://www.instagram.com")
    elif "open chatgpt" in request:
        webbrowser.open("https://chat.openai.com")
    elif "open" in request:
        query = request.replace("open", "").strip()
        pyautogui.press("super")
        pyautogui.typewrite(query)
        pyautogui.sleep(2)
        pyautogui.press("enter")
    elif "wikipedia" in request:
        topic = request.replace("search wikipedia", "").strip()
        try:
            result = wikipedia.summary(topic, sentences=2)
            speak(result)
            update_output("Jarvis: " + result)
        except:
            update_output("Jarvis: Could not find information on Wikipedia.")
    elif "search google" in request:
        query = request.replace("search google", "").strip()
        webbrowser.open(f"https://www.google.com/search?q={query}")
        update_output("Jarvis: Searching on Google...")
    elif "ask ai" in request:
        query = request.replace("ask ai", "").strip()
        update_output("Generating response...")
        response = ai.ask_jarvis(query)
        print(query)
        update_output("Your Response: " + response)
        speak(response)
        print(response)
    else:
        speak("Sorry, I don't understand that.")
        update_output("Jarvis: Sorry, I don't understand that command.")

#GUI Setup
app = tk.Tk()
app.title("Jarvis - AI Voice Assistant")
app.geometry("620x560")
app.resizable(False, False)

label = tk.Label(app, text="Jarvis - Gemini Powered Voice Assistant", font=("Arial", 16))
label.pack(pady=10)

output_box = scrolledtext.ScrolledText(app, wrap=tk.WORD, width=70, height=20, font=("Consolas", 11))
output_box.pack(padx=10, pady=10)

listen_button = tk.Button(app, text="🎙️ Start Listening", font=("Arial", 14), command=lambda: Thread(target=continuous_listen).start())
listen_button.pack(pady=5)

toggle_button = tk.Button(app, text="🌓 Toggle Mode", font=("Arial", 12), command=toggle_theme)
toggle_button.pack(pady=5)

apply_theme()
app.mainloop()
