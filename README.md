# Animating GOES-19 Satellite Imagery with Python 🛰️

## 🛰️ GOES-19 Satellite Latest Northeast Image
![GOES-19 Satellite Latest Northeast Image](https://cdn.star.nesdis.noaa.gov/GOES19/ABI/SECTOR/ne/GEOCOLOR/600x600.jpg)

## 🛰️ Sample Output

![Satellite Animation](https://github.com/DavidTropiansky/Animating-GOES-19-Satellite-Imagery-with-Python/blob/main/animation_2025-06-12_02-26.gif?raw=true)



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

-     20251602341_GOES19-ABI-ne-GEOCOLOR-1200x1200.jpg

That number prefix encodes the timestamp. We extract the hour (`HHMM`) from this to filter images taken between **11:00 and 23:00 UTC**.

### Workflow:

1. Scrape the public directory using `requests` and `BeautifulSoup`
2. Filter images by resolution (`1200x1200`) and timestamp
3. Download the latest 200 matching images directly into memory using `BytesIO`
4. Download images in parallel, preserving the correct order. 
5. Convert each to RGB format with `Pillow`
6. Stitch them into an MP4 video using `imageio`

This approach avoids writing individual image files to disk, improving speed and efficiency.
## 📁 Step 1: Scraping the NOAA Image Directory

The GOES-19 images are hosted at:  
https://cdn.star.nesdis.noaa.gov/GOES19/ABI/SECTOR/ne/GEOCOLOR/

Each image is named with a prefix like `20251602346_GOES19-ABI-ne-GEOCOLOR-1200x1200.jpg`.  
That numeric prefix encodes the time of capture — the last four digits are the time in HHMM format (e.g., 2346 means 11:46 PM UTC).  

Here's how we list the most recent 200 high-res daytime images:

     
     import os
     import re
     import requests
     from bs4 import BeautifulSoup
     from urllib.parse import urljoin

     BASE_URL = "https://cdn.star.nesdis.noaa.gov/GOES19/ABI/SECTOR/ne/GEOCOLOR/"
     MAX_IMAGES = 200
     START_TIME = 900   # 9:00 AM
     END_TIME = 2100    # 9:00 PM

     def list_image_urls():
         resp = requests.get(BASE_URL)
         soup = BeautifulSoup(resp.text, "html.parser")

         img_urls = []
         regex = re.compile(r".*1200x1200\.jpg$")

         for a in soup.find_all('a', href=True):
             href = a['href']
             if not regex.match(href):
                 continue

             match = re.search(r"(\d{4})_GOES19", href)
             if not match:
                 continue

             try:
                 time_val = int(match.group(1)[-4:])
             except ValueError:
                 continue

             if START_TIME <= time_val <= END_TIME:
                 img_urls.append(urljoin(BASE_URL, href))

         return sorted(img_urls)[-MAX_IMAGES:]

---

## ⚡ Step 2: Downloading Images in Parallel  

We use `concurrent.futures` to download images efficiently, while ensuring they remain in chronological order:

     
     from concurrent.futures import ThreadPoolExecutor, as_completed
     from PIL import Image
     from io import BytesIO

     THREADS = 10

     def fetch_image(index, url, total):
         try:
             print(f"Fetching image {index+1}/{total}: {os.path.basename(url)}")
             r = requests.get(url, stream=True, timeout=10)
             img = Image.open(BytesIO(r.content)).convert("RGB")
             return index, img
         except Exception as e:
             print(f"Error fetching {url}: {e}")
             return index, None

---

## 🎬 Step 3: Create the MP4 Animation  

We’ll use `imageio.v3.imwrite()` to write a fast and smooth MP4 video:

     
     import imageio.v3 as iio
     from datetime import datetime

     FPS = 30

     def create_mp4_from_images(images, output_path, fps=FPS):
         images = [img for img in images if img]
         if not images:
             print("No images to animate.")
             return

         print(f"Creating MP4 with {len(images)} frames at {fps} FPS...")
         iio.imwrite(output_path, images, fps=fps)
         print(f"Saved animation to {output_path}")

---

## 🧩 Step 4: Putting It All Together  

     
     def main():
         urls = list_image_urls()
         print(f"Found {len(urls)} images from {START_TIME} to {END_TIME} UTC")

         total = len(urls)
         images = [None] * total

         with ThreadPoolExecutor(max_workers=THREADS) as executor:
             futures = [executor.submit(fetch_image, i, url, total) for i, url in enumerate(urls)]
             for future in as_completed(futures):
                 index, img = future.result()
                 images[index] = img

         timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M")
         output_path = f"GOES19_Northeast_{timestamp}.mp4"
         create_mp4_from_images(images, output_path)

     main()





---

## 🎥 Sample Output

The script generates an MP4 video file named like:

-     animation_20250610_142530.mp4

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

