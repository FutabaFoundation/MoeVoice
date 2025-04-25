# MoeVoice
最初设想：一个程序，在我说话语音识别转文字输入，程序自己润色文案，比如说可以变得正式、或者变得可爱，能在句中加入颜文字
设想代码：
import openai
import whisper
import sounddevice as sd
import numpy as np
import pyperclip
from scipy.io.wavfile import write

# ChatGPT API Key
openai.api_key = '你的OpenAI API密钥'

# 录音设置
def record_audio(duration=5, filename="input.wav"):
    fs = 44100
    print("开始录音，说话吧...")
    audio = sd.rec(int(duration * fs), samplerate=fs, channels=1)
    sd.wait()
    write(filename, fs, audio)
    print("录音完成。")

# Whisper转文字
def transcribe_audio(filename="input.wav"):
    model = whisper.load_model("base")
    result = model.transcribe(filename)
    return result["text"]

# GPT润色
def polish_text(text, style="可爱"):
    style_prompt = {
        "可爱": "请把这段话改得更可爱，加入一些颜文字和撒娇语气：",
        "正式": "请将这段话润色成正式书面语：",
        "搞笑": "请将这段话改写成幽默搞笑风格：",
        "商务": "请将这段话改写为适合商务沟通的语气："
    }
    prompt = style_prompt.get(style, style_prompt["可爱"]) + text
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response['choices'][0]['message']['content']

# 主流程
def main():
    record_audio()
    text = transcribe_audio()
    print("识别原文：", text)

    style = input("想要什么风格（可爱 / 正式 / 搞笑 / 商务）？")
    polished = polish_text(text, style)
    
    print("\n润色结果：", polished)
    pyperclip.copy(polished)
    print("结果已复制到剪贴板~")

main()