#!/usr/bin/env python3

import base64
import os
import sys
from datetime import datetime

import requests

OPENROUTER_API_KEY = ""
OPENROUTER_API_URL = "https://openrouter.ai/api/v1/chat/completions"
# MODEL = "google/gemini-2.5-flash-image-preview"
# MODEL = "google/gemini-3-pro-image-preview"
MODEL = "google/gemini-2.5-flash-image"


def generate_image(prompt: str, image_path: str = None) -> bytes:
    headers = {
        "Authorization": f"Bearer {OPENROUTER_API_KEY}",
        "Content-Type": "application/json",
    }

    content = [{"type": "text", "text": prompt}]

    if image_path:
        with open(image_path, "rb") as f:
            image_data = base64.b64encode(f.read()).decode()
        content.append(
            {
                "type": "image_url",
                "image_url": {"url": f"data:image/png;base64,{image_data}"},
            }
        )

    payload = {
        "model": MODEL,
        "messages": [{"role": "user", "content": content}],
        "modalities": ["image", "text"],
        "image_config": {"aspect_ratio": "16:9"},
    }

    print(f"Sending request to OpenRouter with prompt: {prompt[:50]}...")
    response = requests.post(OPENROUTER_API_URL, json=payload, headers=headers)

    if response.status_code != 200:
        print(f"Error: {response.status_code}")
        print(f"Response: {response.text}")
        sys.exit(1)

    data = response.json()

    if "choices" in data and len(data["choices"]) > 0:
        message = data["choices"][0].get("message", {})
        if "images" in message and len(message["images"]) > 0:
            image_data = message["images"][0]
            image_url = image_data.get("image_url", {}).get("url")
            if image_url:
                if image_url.startswith("data:image/png;base64,"):
                    base64_str = image_url.replace("data:image/png;base64,", "")
                    return base64.b64decode(base64_str)
                elif image_url.startswith("data:image/jpeg;base64,"):
                    base64_str = image_url.replace("data:image/jpeg;base64,", "")
                    return base64.b64decode(base64_str)
                else:
                    img_response = requests.get(image_url)
                    if img_response.status_code == 200:
                        return img_response.content
                    else:
                        print(f"Error downloading image: {img_response.status_code}")
                        sys.exit(1)

    print("No image data in response")
    print(f"Full response: {data}")
    sys.exit(1)


def save_image(image_data: bytes, prompt: str) -> str:
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"generated_image_{timestamp}.png"
    filepath = os.path.join(os.getcwd(), filename)

    with open(filepath, "wb") as f:
        f.write(image_data)

    print(f"Image saved to: {filepath}")
    return filepath


def main():
    if len(sys.argv) < 2:
        print("Usage: python generate_image.py '<prompt>' [-i <image_path>]")
        print("Example: python generate_image.py 'A beautiful sunset over mountains'")
        print("Example: python generate_image.py 'remove the background' -i my_img.png")
        sys.exit(1)

    args = sys.argv[1:]
    prompt_parts = []
    image_path = None

    i = 0
    while i < len(args):
        if args[i] == "-i" and i + 1 < len(args):
            image_path = args[i + 1]
            i += 2
        else:
            prompt_parts.append(args[i])
            i += 1

    prompt = " ".join(prompt_parts)
    print(f"Generating image for prompt: {prompt}")
    if image_path:
        print(f"Using context image: {image_path}")

    image_data = generate_image(prompt, image_path)
    save_image(image_data, prompt)
    print("Done!")


if __name__ == "__main__":
    main()
