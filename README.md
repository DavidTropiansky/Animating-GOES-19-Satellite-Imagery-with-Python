# Animating GOES-19 Satellite Imagery with Python 🛰️

![GOES-19 Satellite](https://cdn.star.nesdis.noaa.gov/GOES19/ABI/SECTOR/ne/GEOCOLOR/600x600.jpg)

## 🌎 Overview

This project demonstrates how to fetch the latest **GOES-19 satellite imagery** from NOAA and create a **high-resolution MP4 animation** showing weather changes over the northeastern United States. The animation is built using `Pillow`, `requests`, `BeautifulSoup`, and `imageio`.

We specifically:
- Download the **most recent 200 satellite images** (1200×1200 resolution - can also download 600x600 or 300x300). Assuming you don't filter the captured times, and use a 24 hour day, 1 day ≈ 288 images, 2 days ≈ 576 images, 3 days ≈ 864 images, 1 week ≈ 2016 images, as one image represents 5 minutes. 
- For this code, we are choosing to filter out nighttime, so we only look at images captured **between 11:00 and 23:00 UTC (THIS IS OPTIONAL)**
- Animate them into a smooth **MP4 video at 30 FPS**
- Automatically timestamp the output file

> 🔁 All image downloads and animations happen **in-memory** for speed — no disk writing of intermediate files.

---

## 📦 Requirements

Install dependencies with pip:
-     pip install requests beautifulsoup4 pillow imageio

## 🚀 Usage

Run the script from your terminal:

-     python animate_goes19.py

This will:

- Scrape the most recent 1200×1200 satellite images from [NOAA's GOES-19 archive](https://cdn.star.nesdis.noaa.gov/GOES19/ABI/SECTOR/ne/GEOCOLOR/)
  - **Number of images per time span** (5 min per image):
    - 1 day ≈ 288 images
    - 2 days ≈ 576 images
    - 3 days ≈ 864 images
    - 1 week ≈ 2016 images
- Filter them to only include images taken between **07:00 and 20:00 UTC**
- Limit to the **latest 200 images**
- Create an **MP4 video** (`animation_YYYYMMDD_HHMMSS.mp4`) at **30 FPS**

---

## 🔍 How It Works

NOAA stores their images with filenames like:

-     
     20251602341_GOES19-ABI-ne-GEOCOLOR-1200x1200.jpg

That number prefix encodes the timestamp. We extract the hour (`HHMM`) from this to filter images taken between **11:00 and 23:00 UTC**.

### Workflow:

1. Scrape the public directory using `requests` and `BeautifulSoup`
2. Filter images by resolution (`1200x1200`) and timestamp
3. Download the latest 200 matching images directly into memory using `BytesIO`
4. Convert each to RGB format with `Pillow`
5. Stitch them into an MP4 video using `imageio`

This approach avoids writing individual image files to disk, improving speed and efficiency.

---

## 🎥 Sample Output

The script generates an MP4 video file named like:

-     
     animation_20250610_142530.mp4

This video plays back the satellite images as a smooth time-lapse animation of the northeastern US from GOES-19.

---

## 🧮 Data Timing and Image Counts

- Each image corresponds to a 5-minute interval.
- Approximate image counts for time spans:

  - 1 day ≈ 288 images
  - 2 days ≈ 576 images
  - 3 days ≈ 864 images
  - 1 week ≈ 2016 images

The script limits downloads to the most recent 200 images to keep the animation manageable.

---

## ⚙️ Requirements

- Python 3.7+
- Packages:

  - requests
  - beautifulsoup4
  - pillow
  - imageio[v3]

Install with:

-     pip install requests beautifulsoup4 pillow imageio[v3]

---

## 📄 Demo

You can run this project in [Google Colab](https://colab.research.google.com/drive/1AZ7UEbseWTCa9bMT_OxUv6HA0YB7a9SD?usp=sharing).  
I left a choice of using 600px or 1200px images (higher resolution = more time/space).


---

## 👤 Author

Created by David Tropiansky

---

Feel free to open issues or submit pull requests for improvements!
