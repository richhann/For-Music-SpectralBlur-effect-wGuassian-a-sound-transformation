Juypter Notebook from Anaconda

Spectral Blur - a reverb-like texturalizer that adds  an ethereal spectral wash to any sound.
![image](https://github.com/richhann/SpectalBlurwGuassian/assets/63624702/c26b02d4-281a-416d-b8ce-2df6827f7c45)

Using Scripy UI for quick prototype

<img width="327" alt="image" src="https://github.com/richhann/SpectalBlurwGuassian/assets/63624702/6e4c60d2-bf37-449e-9f76-687ec1fddad7">



# Code
```python
import os
import numpy as np
import tkinter as tk
from tkinter import filedialog, messagebox
from threading import Thread
from scipy.io.wavfile import read, write
from scipy.fft import fft, ifft
from scipy.ndimage import gaussian_filter1d

# Constants
INPUT_DIR = 'sound'
OUTPUT_DIR = 'spectral_sound'
BLEND_FACTOR = 0.5  # Default blend factor
FILE_PREFIX = 'processed_'  # Default file prefix

# Ensure output directory exists
if not os.path.exists(OUTPUT_DIR):
    os.makedirs(OUTPUT_DIR)

def spectral_blend(file_path, blend_factor):
    """Apply spectral blur effect to a .wav file."""
    rate, data = read(file_path)
    if data.ndim == 2:
        data = data.mean(axis=1)  # Convert to mono if stereo
    
    # Perform FFT
    spectrum = fft(data)
    
    # Apply Gaussian blur to the magnitude of the spectrum
    magnitude = np.abs(spectrum)
    phase = np.angle(spectrum)
    blurred_magnitude = gaussian_filter1d(magnitude, sigma = blend_factor * len(magnitude))
    
    # Reconstruct the blurred spectrum
    blurred_spectrum = blurred_magnitude * np.exp(1j * phase)
    
    # Perform inverse FFT
    blurred_data = np.real(ifft(blurred_spectrum))
    
    # Normalize the processed data
    blurred_data = np.int16(blurred_data / np.max(np.abs(blurred_data)) * 32767)
    
    return rate, blurred_data

def process_files():
    if not os.path.exists(INPUT_DIR):
        messagebox.showwarning("Input Directory Not Found", f"The directory '{INPUT_DIR}' does not exist.")
        return
    
    if not OUTPUT_DIR:
        messagebox.showwarning("Output Directory Not Set", "Please set the output directory first.")
        return
    
    files = [f for f in os.listdir(INPUT_DIR) if f.endswith('.wav')]
    
    for idx, file_name in enumerate(files):
        file_path = os.path.join(INPUT_DIR, file_name)
        rate, processed_data = spectral_blend(file_path, BLEND_FACTOR)
        
        output_file_name = f"{FILE_PREFIX}{idx+1}.wav"
        output_file_path = os.path.join(OUTPUT_DIR, output_file_name)
        
        write(output_file_path, rate, processed_data)
        print(f"Processed {file_name} -> {output_file_name}")

    messagebox.showinfo("Processing Complete", "Files have been processed and saved.")

def select_files():
    global INPUT_DIR
    files = filedialog.askopenfilenames(filetypes=[("WAV files", "*.wav")])
    if files:
        INPUT_DIR = os.path.dirname(files[0])
        print(f"Input directory set to {INPUT_DIR}")
        process_files_thread = Thread(target=process_files)
        process_files_thread.start()

def select_output_dir():
    global OUTPUT_DIR
    OUTPUT_DIR = filedialog.askdirectory()
    if OUTPUT_DIR:
        print(f"Output directory set to {OUTPUT_DIR}")

def start_processing():
    """Function to start the processing of files in a separate thread."""
    process_files_thread = Thread(target=process_files)
    process_files_thread.start()

def go_button(frame):
    """Function to add a 'GO' button to the GUI."""
    tk.Button(frame, text="GO", command=start_processing).grid(row=4, columnspan=2, pady=(10, 0))

def update_blend_factor(value):
    global BLEND_FACTOR
    BLEND_FACTOR = float(value)
    print(f"Blend factor set to {BLEND_FACTOR}")

def update_file_prefix(prefix):
    global FILE_PREFIX
    FILE_PREFIX = prefix
    print(f"File prefix set to {FILE_PREFIX}")

def create_gui():
    root = tk.Tk()
    root.title("Spectral Sound Processor")
    
    frame = tk.Frame(root, padx=10, pady=10)
    frame.pack(padx=10, pady=10)

    tk.Label(frame, text="Select .wav files to process:").grid(row=0, column=0, pady=(0, 10))
    tk.Button(frame, text="Select Files", command=select_files).grid(row=0, column=1, pady=(0, 10))

    tk.Label(frame, text="Set Output Directory:").grid(row=1, column=0, pady=(10, 0))
    tk.Button(frame, text="Select Output Directory", command=select_output_dir).grid(row=1, column=1, pady=(10, 0))

    tk.Label(frame, text="Blend Factor:").grid(row=2, column=0, pady=(10, 0))
    blend_slider = tk.Scale(frame, from_=0.0, to=1.0, resolution=0.01, orient=tk.HORIZONTAL, command=update_blend_factor)
    blend_slider.set(BLEND_FACTOR)
    blend_slider.grid(row=2, column=1, pady=(10, 0))

    tk.Label(frame, text="File Prefix:").grid(row=3, column=0, pady=(10, 0))
    file_prefix_entry = tk.Entry(frame)
    file_prefix_entry.insert(0, FILE_PREFIX)
    file_prefix_entry.grid(row=3, column=1, pady=(10, 0))
    file_prefix_entry.bind("<KeyRelease>", lambda event: update_file_prefix(file_prefix_entry.get()))

    go_button(frame)
    
    root.mainloop()

if __name__ == "__main__":
    create_gui()

```
