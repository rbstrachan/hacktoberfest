!pip install ipywidgets ollama
!apt-get update && apt-get install -y zstd
!curl -fsSL https://ollama.com/install.sh | sh

import json
import subprocess
import time
import ipywidgets as widgets
from IPython.display import display

print("starting background server for AI model...")
subprocess.Popen(["ollama", "serve"])
time.sleep(4)

print("downloading Llama 3.2 (1B) model...")
subprocess.run(["ollama", "pull", "llama3.2:1b"], check=True)
print("model downloaded succesfully")

input_text = widgets.Textarea(
    value="We need to wrap up this project by Friday so we don't end up back at square one.",
    placeholder="Paste your difficult English sentence or phrase here...",
    description="English:",
    layout=widgets.Layout(width="95%", height="80px")
)

analyze_btn = widgets.Button(
    description="Explain Phrase →",
    button_style="success",
    icon="language",
    layout=widgets.Layout(width="200px")
)

status_output = widgets.Output()

eng_output = widgets.HTML(value="<i>English explanation will appear here...</i>")
fr_output = widgets.HTML(value="<i>Explication française s'affichera ici...</i>")

tabs = widgets.Tab(children=[eng_output, fr_output])
tabs.set_title(0, "🇬🇧 English")
tabs.set_title(1, "🇫🇷 Français")

def analyze_phrase(b):
    text = input_text.value.strip()
    if not text:
        return
    
    with status_output:
        status_output.clear_output()
        print("Please wait. Analyzing phrase with Llama 3.2...")

    prompt = f"""
    Analyze this English excerpt for an intermediate English speaker who is a native French speaker:
    "{text}"

    Respond ONLY with valid JSON using this EXACT structure (no markdown, no backticks):
    {{
        "english": {{
            "meaning": "1-sentence explanation of meaning or idiom in plain English.",
            "alternatives": "Two natural alternative ways to say this in English.",
            "pronunciation": "pronunciation notes for difficult words"
        }},
        "french": {{
            "meaning": "Explication claire en 1 phrase du sens en français.",
            "alternatives": "Deux façons alternatives et naturelles de le dire en anglais.",
            "pronunciation": "Notes de prononciation pour les mots difficiles."
        }}
    }}
    """

    try:
        import ollama
        response = ollama.generate(model="llama3.2:1b", prompt=prompt, format="json")
        data = json.loads(response["response"])

        eng_output.value = f"""
        <div style="padding: 10px; line-height: 1.6;">
            <p><b>Meaning:</b> {data['english']['meaning']}</p>
            <p><b>Alternatives:</b> {data['english']['alternatives']}</p>
            <p><b>Pronunciation:</b> {data['english']['pronunciation']}</p>
        </div>
        """
        fr_output.value = f"""
        <div style="padding: 10px; line-height: 1.6;">
            <p><b>Sens:</b> {data['french']['meaning']}</p>
            <p><b>Alternatives:</b> {data['french']['alternatives']}</p>
            <p><b>Prononciation:</b> {data['french']['pronunciation']}</p>
        </div>
        """

        with status_output:
            status_output.clear_output()
            print("✅ Done!")

    except Exception as e:
        with status_output:
            status_output.clear_output()
            print(f"❌ Error: {e}")

analyze_btn.on_click(analyze_phrase)
display(input_text, analyze_btn, status_output, tabs)