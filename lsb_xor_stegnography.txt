import cv2
import os

# XOR encryption and decryption
def xor_encrypt(text, key):
    return ''.join(chr(ord(c) ^ ord(key[i % len(key)])) for i, c in enumerate(text))

def xor_decrypt(cipher, key):
    return ''.join(chr(ord(c) ^ ord(key[i % len(key)])) for i, c in enumerate(cipher))

# Convert JPG to PNG for lossless steganography
def convert_to_png(jpg_path, png_path):
    image = cv2.imread(jpg_path)
    if image is None:
        print("❌ Cannot read JPG image!")
        return False
    cv2.imwrite(png_path, image)
    print(f"[📸] Converted '{jpg_path}' to '{png_path}'")
    return True

# Encode message in image using LSB
def hide_message(image_path, message, key, output_path):
    encrypted = xor_encrypt(message, key) + "#END#"  # End marker
    binary_data = ''.join(format(ord(char), '08b') for char in encrypted)

    image = cv2.imread(image_path)
    if image is None:
        print("❌ Image not found!")
        return

    rows, cols, _ = image.shape
    capacity = rows * cols

    if len(binary_data) > capacity:
        print("❌ Message too long for this image!")
        return

    index = 0
    for i in range(rows):
        for j in range(cols):
            if index < len(binary_data):
                pixel = image[i, j]
                pixel[0] = (pixel[0] & ~1) | int(binary_data[index])  # Change blue channel LSB
                image[i, j] = pixel
                index += 1
            else:
                break
        if index >= len(binary_data):
            break

    cv2.imwrite(output_path, image, [cv2.IMWRITE_PNG_COMPRESSION, 0])
    print(f"[✅] Message hidden in '{output_path}'")

# Decode and decrypt message from image
def reveal_message(image_path, key):
    image = cv2.imread(image_path)
    if image is None:
        return "❌ Image not found!"

    binary_data = ''
    message = ''

    rows, cols, _ = image.shape
    for i in range(rows):
        for j in range(cols):
            binary_data += str(image[i, j, 0] & 1)
            if len(binary_data) % 8 == 0:
                char = chr(int(binary_data[-8:], 2))
                message += char
                if "#END#" in message:
                    break
        if "#END#" in message:
            break

    encrypted = message.replace("#END#", "")
    decrypted = xor_decrypt(encrypted, key)
    return decrypted

# === Main Program ===
if __name__ == "__main__":
    jpg_path = "/mnt/data/input.jpg"
    png_path = "/mnt/data/input.png"
    output_image = "/mnt/data/output_stego.png"
    secret_text = "Results will be announced next week."
    encryption_key = "lab2025"

    if convert_to_png(jpg_path, png_path):
        hide_message(png_path, secret_text, encryption_key, output_image)
        result = reveal_message(output_image, encryption_key)
        print("🔓 Decrypted message:", result)
