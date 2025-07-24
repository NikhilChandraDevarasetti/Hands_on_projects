import cv2
import pytesseract
import numpy as np
from PIL import Image
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor

def detect_table_cells(image_path):
    image = cv2.imread(image_path)
    original = image.copy()
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

    # Preprocessing
    _, binary = cv2.threshold(~gray, 150, 255, cv2.THRESH_BINARY)

    # Horizontal lines
    horizontal = binary.copy()
    hor_size = horizontal.shape[1] // 20
    hor_structure = cv2.getStructuringElement(cv2.MORPH_RECT, (hor_size, 1))
    horizontal = cv2.erode(horizontal, hor_structure)
    horizontal = cv2.dilate(horizontal, hor_structure)

    # Vertical lines
    vertical = binary.copy()
    ver_size = vertical.shape[0] // 20
    ver_structure = cv2.getStructuringElement(cv2.MORPH_RECT, (1, ver_size))
    vertical = cv2.erode(vertical, ver_structure)
    vertical = cv2.dilate(vertical, ver_structure)

    mask = horizontal + vertical

    # Detect contours (cells)
    contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
    cells = []
    for c in contours:
        x, y, w, h = cv2.boundingRect(c)
        if w > 40 and h > 15:
            cells.append((x, y, w, h))

    # Sort top to bottom, then left to right within rows
    cells = sorted(cells, key=lambda b: (b[1], b[0]))

    return original, cells

def group_cells_by_rows(cells, tolerance=10):
    rows = []
    current_row = []
    last_y = None
    for cell in cells:
        x, y, w, h = cell
        if last_y is None or abs(y - last_y) < tolerance:
            current_row.append(cell)
        else:
            rows.append(sorted(current_row, key=lambda b: b[0]))
            current_row = [cell]
        last_y = y
    if current_row:
        rows.append(sorted(current_row, key=lambda b: b[0]))
    return rows

def extract_cell_data(image, rows):
    table_data = []
    for row in rows:
        row_data = []
        for (x, y, w, h) in row:
            crop = image[y:y+h, x:x+w]
            text = pytesseract.image_to_string(crop, config='--psm 7').strip()
            b, g, r = np.mean(crop, axis=(0, 1))
            row_data.append((text, (int(r), int(g), int(b))))
        table_data.append(row_data)
    return table_data

def build_ppt_from_data(table_data, output_file):
    prs = Presentation()
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    row_count = len(table_data)
    col_count = max(len(row) for row in table_data)

    table = slide.shapes.add_table(row_count, col_count, Inches(0.3), Inches(0.5), Inches(9), Inches(5.5)).table

    for i, row in enumerate(table_data):
        for j, (text, color) in enumerate(row):
            if j >= col_count: continue
            cell = table.cell(i, j)
            cell.text = text
            para = cell.text_frame.paragraphs[0]
            para.font.size = Pt(9)
            cell.fill.solid()
            cell.fill.fore_color.rgb = RGBColor(*color)

            if (color[0] + color[1] + color[2]) < 380:  # dark bg
                para.font.color.rgb = RGBColor(255, 255, 255)

    prs.save(output_file)
    print(f"✅ Saved to: {output_file}")

# ---- USAGE ----
image_path = "your_table_image.png"
output_pptx = "table_output_from_image.pptx"

img, cells = detect_table_cells(image_path)
rows = group_cells_by_rows(cells)
data = extract_cell_data(img, rows)
build_ppt_from_data(data, output_pptx)
