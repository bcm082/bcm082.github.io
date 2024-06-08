---
title: "Barcode_qrcode_generator"
date: 2024-06-08T12:02:57-04:00
draft: true
---

# Create Your Own Barcode and QR Code Generator with Streamlit

In this tutorial on building your own Barcode and QR Code Generator using Python and Streamlit! This step-by-step guide will walk you through the process of creating a simple yet powerful application to generate barcodes and QR codes. Whether you're a beginner or have some coding experience, you'll find this tutorial easy to follow. Let's get started!

## Step 1: Setting Up Your Environment

First, make sure you have Python installed on your computer. You will also need to install the required libraries. Open your terminal or command prompt and run the following commands:

```pip install streamlit python-barcode qrcode[pil] pillow```

These libraries will help us create the web application and generate the barcodes and QR codes.

## Step 2: Writing the Code

Create a new Python file (e.g., barcode_qrcode_generator.py) and open it in your favorite code editor. We'll start by importing the necessary libraries.

```python
import streamlit as st
import barcode
from barcode.writer import SVGWriter
from io import BytesIO
import base64
import qrcode
from qrcode.image.svg import SvgImage
from zipfile import ZipFile
```

## Step 3: Creating the Barcode Generator Function

Next, we'll write a function to generate barcodes from UPC (Universal Product Code) numbers. This function will validate the input and generate the barcode if the input is valid.

```python
def generate_barcode(upc):
    if not upc.isdigit() or len(upc) != 12:
        return None, "Invalid UPC"
    upc_barcode = barcode.get('upc', upc, writer=SVGWriter())
    buffer = BytesIO()
    upc_barcode.write(buffer)
    buffer.seek(0)
    return buffer, None
```

- Input Validation: Checks if the input is a 12-digit number.
- Barcode Creation: Generates the barcode and writes it to a buffer.

## Step 4: Creating the QR Code Generator Function

We'll also create a function to generate QR codes from user-provided data (text or URL).

```python
def generate_qrcode(data):
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )
    qr.add_data(data)
    qr.make(fit=True)
    img = qr.make_image(image_factory=SvgImage)
    buffer = BytesIO()
    img.save(buffer)
    buffer.seek(0)
    return buffer
```
- QR Code Configuration: Sets the parameters for the QR code.
- Data Addition: Adds the user-provided data to the QR code.
- Buffer Handling: Saves the QR code image to a buffer.

## Step 5: Displaying the Codes

We'll create a helper function to convert the buffer content into an image tag that can be displayed in the web interface.

```python
def get_image_tag(buffer):
    data_uri = base64.b64encode(buffer.read()).decode('utf-8')
    img_tag = f'<img src="data:image/svg+xml;base64,{data_uri}" alt="Barcode/QR Code"/>'
    return img_tag
```
- Base64 Encoding: Encodes the buffer content in base64.
- Image Tag Creation: Generates an HTML image tag with the encoded data.

## Step 6: Building the Streamlit Interface

Now we'll set up the Streamlit interface, allowing users to interact with the application.

```python
st.title("Barcode and QR Code Generator")
code_type = st.radio("Select code type:", ("Barcode", "QR Code"))

if code_type == "Barcode":
    input_label = "Enter a 12-digit UPC number (or multiple UPCs separated by commas):"
else:
    input_label = "Enter URL or text for QR code:"

user_input = st.text_input(input_label, "")
```
- Title: Sets the title of the web app.
- Code Type Selection: Allows users to select whether they want to generate a barcode or a QR code.
- User Input: Prompts users to enter the necessary input based on their selection.

## Step 7: Generating and Downloading Codes

We'll add functionality to generate and download the barcodes or QR codes based on the user's input.

```python
if st.button(f"Generate {code_type}"):
    if code_type == "Barcode":
        upcs = [upc.strip() for upc in user_input.split(',')]
        valid_buffers = []
        errors = []
        for upc in upcs:
            buffer, error = generate_barcode(upc)
            if buffer:
                valid_buffers.append((buffer, upc))
            if error:
                errors.append(f"UPC {upc}: {error}")

        if len(valid_buffers) > 1:
            zip_buffer = BytesIO()
            with ZipFile(zip_buffer, 'w') as zip_file:
                for buffer, upc in valid_buffers:
                    zip_file.writestr(f"{upc}.svg", buffer.getvalue())
            zip_buffer.seek(0)
            st.download_button(label="Download ZIP",
                               data=zip_buffer,
                               file_name="barcodes.zip",
                               mime="application/zip")
        elif valid_buffers:
            buffer, upc = valid_buffers[0]
            st.download_button(label=f"Download Barcode",
                               data=buffer,
                               file_name=f"{upc}.svg",
                               mime="image/svg+xml")
            buffer.seek(0)
            st.markdown(get_image_tag(buffer), unsafe_allow_html=True)

        if errors:
            st.error("\n".join(errors))

    else:
        buffer = generate_qrcode(user_input)
        st.download_button(label=f"Download QR Code",
                           data=buffer,
                           file_name=f"{user_input}.svg",
                           mime="image/svg+xml")
        buffer.seek(0)
        st.markdown(get_image_tag(buffer), unsafe_allow_html=True)
```
- Generate Button: When clicked, it triggers the code generation process.
- Barcode Handling: Processes multiple UPCs, generates barcodes, and handles errors.
- QR Code Handling: Generates a QR code from the input data.
- Download Options: Provides download buttons for single or multiple files.

## Step 8: Adding a Support Button

At the bottom of the page, we can add a "Buy Me a Coffee" button to support the developer.

```python
st.markdown(
    """
    <a href="https://buymeacoffee.com/brunocm" target="_blank">
        <button style="color: black; background-color: #FFDD00; border: none; padding: 10px 20px; text-align: center; display: inline-block; font-size: 16px; margin-top: 20px;">
            Buy Me a Coffee ☕
        </button>
    </a>
    """,
    unsafe_allow_html=True
)
```
## Step 9: Running the Application

To run the application, navigate to the directory where your Python file is located and run the following command:

```streamlit run barcode_qrcode_generator.py```

This command will launch the Streamlit application in your default web browser. You can now generate barcodes and QR codes by entering the required input and clicking the "Generate" button.

## Conclusion

Congratulations! You've successfully created a Barcode and QR Code Generator using Streamlit. This application can generate both barcodes from UPC numbers and QR codes from text or URLs. You can easily download the generated codes as SVG files, making it perfect for various applications.

Feel free to customize and expand this project to suit your needs. Happy coding, and don't forget to grab yourself a coffee!

If you found this tutorial helpful, consider supporting me by buying me a coffee. Enjoy your new barcode and QR code generator!