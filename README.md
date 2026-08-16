import gradio as gr
import torch
import soundfile as sf
from parler_tts import ParlerTTSForConditionalGeneration
from transformers import AutoTokenizer

MODEL_ID = "ai4bharat/indic-parler-tts"

# Device
device = "cuda" if torch.cuda.is_available() else "cpu"

print("Loading Hindi AI Voice Model...")
print("Device:", device)

model = ParlerTTSForConditionalGeneration.from_pretrained(
    MODEL_ID,
    torch_dtype=torch.float16 if device == "cuda" else torch.float32
).to(device)

tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)

description_tokenizer = AutoTokenizer.from_pretrained(
    model.config.text_encoder._name_or_path
)

def generate_voice(text, style):
    if not text or not text.strip():
        raise gr.Error("पहले Hindi text लिखिए।")

    description = (
        "A natural Indian Hindi speaker delivers the speech clearly, "
        "with a warm, confident and expressive voice. "
        "The speaking speed is moderate, pronunciation is clear, "
        "and the recording has high audio quality with no background noise. "
        + style
    )

    # Voice description
    description_inputs = description_tokenizer(
        description,
        return_tensors="pt"
    ).to(device)

    # Hindi text
    prompt_inputs = tokenizer(
        text,
        return_tensors="pt"
    ).to(device)

    with torch.no_grad():
        generation = model.generate(
            input_ids=description_inputs.input_ids,
            attention_mask=description_inputs.attention_mask,
            prompt_input_ids=prompt_inputs.input_ids,
            prompt_attention_mask=prompt_inputs.attention_mask
        )

    audio = generation.cpu().numpy().squeeze()

    output_file = "hindi_ai_voice.wav"

    sf.write(
        output_file,
        audio,
        model.config.sampling_rate
    )

    return output_file


# -------------------------
# Website UI
# -------------------------

with gr.Blocks(
    title="Hindi AI Voice Studio",
    theme=gr.themes.Soft()
) as demo:

    gr.Markdown(
        """
        # 🎙️ Hindi AI Voice Studio
        ### Natural Hindi AI Voice Generator

        **Text लिखिए → Generate Voice → सुनिए → Download कीजिए**
        """
    )

    with gr.Row():

        with gr.Column(scale=2):

            text_input = gr.Textbox(
                label="Hindi Script",
                placeholder="यहाँ अपनी Hindi script लिखिए...",
                lines=10
            )

            style_input = gr.Textbox(
                label="Voice Style",
                value="The speaker sounds confident and motivational.",
                lines=3
            )

            generate_button = gr.Button(
                "🎙️ Generate Hindi Voice",
                variant="primary"
            )

        with gr.Column(scale=1):

            audio_output = gr.Audio(
                label="Generated Voice",
                type="filepath"
            )

    generate_button.click(
        fn=generate_voice,
        inputs=[text_input, style_input],
        outputs=audio_output
    )

    gr.Markdown(
        """
        ---
        **Project:** YouTube AI Voice Studio  
        **Language:** Hindi 🇮🇳  
        **Use:** YouTube Shorts / Testing
        """
    )


if __name__ == "__main__":
    demo.launch()
