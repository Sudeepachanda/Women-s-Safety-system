# Women-s-Safety-system
Advanced self-intelligence sound-based women's safety system, where speech converted to text using speech recognition algorithm by Pysound and recognition of harmful words from the dictionary using NLP

**SOURCE CODE**

import pyaudio
import wave
import librosa
import numpy as np
import speech_recognition as sr
import pyttsx3
import serial
ser = serial.Serial('COM6', 9600)
def record_audio(filename="output.wav", duration=5):
    CHUNK = 1024
    FORMAT = pyaudio.paInt16
    CHANNELS = 1
    RATE = 44100

    p = pyaudio.PyAudio()

    stream = p.open(format=FORMAT,
                    channels=CHANNELS,
                    rate=RATE,
                    input=True,
                    frames_per_buffer=CHUNK)

    print("Recording...")

    frames = []

    for i in range(0, int(RATE / CHUNK * duration)):
        data = stream.read(CHUNK)
        frames.append(data)

    print("Finished recording")

    stream.stop_stream()
    stream.close()
    p.terminate()

    wf = wave.open(filename, 'wb')
    wf.setnchannels(CHANNELS)
    wf.setsampwidth(p.get_sample_size(FORMAT))
    wf.setframerate(RATE)
    wf.writeframes(b''.join(frames))
    wf.close()
def dyslexia_detection(audio_file):
    y, sr = librosa.load(audio_file, sr=None)

    # Extract mean MFCC values
    mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=13)
    mean_mfccs = np.mean(mfccs, axis=1)



    # Compare mean MFCC values to the threshold
    print(np.mean(mean_mfccs))
    le=(np.mean(mean_mfccs)*-1)-10
    le=100-(le*10)
    print(le)
    return le

def speech_to_text(audio_file):
    recognizer = sr.Recognizer()

    with sr.AudioFile(audio_file) as source:
        audio_data = recognizer.record(source)

    try:
        text = recognizer.recognize_google(audio_data)
        return text
    except sr.UnknownValueError:
        print("Speech Recognition could not understand audio")
        return ""
    except sr.RequestError as e:
        print(f"Could not request results from Google Speech Recognition service; {e}")
        return ""

def text_to_speech(text):
    engine = pyttsx3.init()
    engine.say(text)
    engine.runAndWait()

if __name__ == "__main__":
    # Record audio
    record_audio()

    # Perform dyslexia detection
    audio_file = "output.wav"
    

    # Convert speech to text
    transcribed_text = speech_to_text(audio_file)
    print("Transcribed Text:", transcribed_text)
    if "help" in transcribed_text:
        result = dyslexia_detection(audio_file)
        print(result)
        if result <-10:
            print("Sending data to MCU")
            ser.write(b'A')
        else:
            ser.write(b'B')
        
    # Display result
    

