# ImageToGoogleSheet

Paint any image on a Google Sheet with Python, one cell per pixel.

The script reads an image with Pillow, adds a new sheet sized to the image, makes the cells square, and then colors every cell with the color of its pixel, using a single Google Sheets API `batchUpdate` request.

It was written for my talk **"Amaze Your Friends by Painting Images on Google Sheets Using Python"** at PyCon India 2023 in Hyderabad.

- Blog post (full walkthrough): BLOG_POST_URL
- Slides (PDF): https://drive.google.com/file/d/17NIxo-QIkDKx-RBlK7R56DHt8upV67Q0/view
- Proposal: https://in.pycon.org/cfp/pycon-india-2023/proposals/amaze-your-friends-by-painting-images-on-google-sheets-using-python~eER80/
- Demo video: https://youtu.be/H2J7bXMvqL4

## How it works

1. **Read the pixels.** Pillow opens the image, converts it to RGB if needed, and NumPy turns the pixel data into an array.
2. **Add a canvas.** A new sheet is added to your spreadsheet with one row per pixel of height and one column per pixel of width. Its name is the month, weekday and Unix timestamp, so repeat runs never collide.
3. **Make the cells square.** Every column is set to 21 pixels, which matches the default row height.
4. **Paint.** The pixels are described as dataclasses (`RequestDataClasses.py`), converted to JSON, and sent as one `updateCells` request.

## Files

| File | Job |
|---|---|
| `ImageToGoogleSheet.py` | Main script: loads the image, adds the sheet, sends the pixels |
| `ImageToJSON.py` | Turns the pixel array into one `updateCells` request |
| `RequestDataClasses.py` | Dataclasses that describe the request (`dataclasses-json` turns them into JSON) |
| `ImageToGoogleSheet.ipynb` | The same steps as a Jupyter notebook (the demo from the talk) |
| `images/` | Small sample images (200 x 200 pixels or close) |

## Setup

### 1. Google Cloud

1. Create a Google Cloud project.
2. Enable the **Google Sheets API** (APIs & Services > Enable APIs and Services).
3. Create a **service account** (APIs & Services > Credentials).
4. Create a **JSON key** for it (service account > Keys > Add Key) and save it next to the script as `key.json`.
5. Share your target spreadsheet with the service account's email address (it ends in `iam.gserviceaccount.com`) as an **Editor**.

> **Keep `key.json` private.** It gives access to anything the service account can edit. `.gitignore` already excludes `key.json` and `*.log`; never commit or share the key.

### 2. Python

Python 3.9 or later is required.

```bash
pip install pillow numpy google-api-python-client google-auth google-auth-httplib2 httplib2 dataclasses-json tqdm colorama
```

### 3. Tell the script which spreadsheet to use

Set the `SPREADSHEETID` environment variable to your spreadsheet's ID. It is the part of the URL after `/d/`:

```
https://docs.google.com/spreadsheets/d/<spreadsheetId>/edit#gid=<sheetId>
```

macOS / Linux:

```bash
export SPREADSHEETID=<spreadsheetId>
```

Windows (PowerShell):

```powershell
$env:SPREADSHEETID = "<spreadsheetId>"
```

## Run

```bash
python ImageToGoogleSheet.py images/monasmall.png
```

A new sheet appears at the bottom of your spreadsheet with the painted image. The image is bigger than most screens, so zoom the sheet out (about 33 percent for a 200 x 200 image) to see all of it.

A log of every request is written to `imagetogogglesheet.log`. It gets large for big images, but it is useful when a request is rejected.

## Notes and limits

- **Start small.** One pixel is one cell and one entry in the request, so large images mean very large requests. The sample images are 200 x 200 pixels (40,000 cells).
- **Transparency is ignored.** RGBA images are converted to RGB.
- **Debugging proxy.** `UseProxy` in `ImageToGoogleSheet.py` is `False` by default. If you switch it on, traffic goes through a local proxy at `127.0.0.1:8888` with certificate checking turned off. Leave it off unless you are inspecting the API calls yourself.

## License

The source code is released under the **GNU General Public License v3.0**; see [LICENSE](LICENSE). In short: you can use, study, change and share it, and if you distribute a modified version you must release it under the GPL too.

The GPL covers the source code only. The sample images in `images/` are for demonstration and keep their own copyright status. The PyCon India logo (`pyconv.png`) belongs to PyCon India and its owners.

## Author

Shashi Jeevan M. P. - [shashijeevan.com](https://shashijeevan.com) - [truepythoneer.com](https://truepythoneer.com)
