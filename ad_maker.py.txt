import sys
from PyQt6.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QPushButton, QColorDialog, QLineEdit, 
                             QLabel, QFileDialog, QFrame)
from PyQt6.QtGui import QFont
from PyQt6.QtCore import Qt
from PIL import Image, ImageDraw, ImageFont

class PhotoshopAdClone(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Photoshop Ad Designer Clone - Laptop Edition")
        self.setGeometry(100, 100, 1100, 700)
        
        self.bg_color = "#1a1a1a"
        self.headline_color = "#ffffff"
        self.subheading_color = "#ffcc00"
        
        self.init_ui()

    def init_ui(self):
        main_widget = QWidget()
        self.setCentralWidget(main_widget)
        h_layout = QHBoxLayout(main_widget)
        
        tools_frame = QFrame()
        tools_frame.setFixedWidth(350)
        tools_frame.setStyleSheet("background-color: #252526; border-radius: 8px; padding: 10px;")
        tools_layout = QVBoxLayout(tools_frame)
        tools_layout.setSpacing(15)
        
        title = QLabel("🎨 Photoshop Ad Control")
        title.setFont(QFont("Arial", 14, QFont.Weight.Bold))
        title.setStyleSheet("color: white;")
        tools_layout.addWidget(title)
        
        tools_layout.addWidget(QLabel("<font color='white'>১. ব্যাকগ্রাউন্ড কালার:</font>"))
        self.btn_bg_color = QPushButton("🎨 কালার সিলেক্ট করুন")
        self.btn_bg_color.setStyleSheet(self.btn_style("#3e3e42"))
        self.btn_bg_color.clicked.connect(self.select_bg_color)
        tools_layout.addWidget(self.btn_bg_color)
        
        tools_layout.addWidget(QLabel("<font color='white'>২. প্রধান শিরোনাম (Headline):</font>"))
        self.input_headline = QLineEdit("💥 ধামাকা অফার! ২০% ছাড়")
        self.input_headline.setStyleSheet(self.input_style())
        self.input_headline.textChanged.connect(self.update_preview)
        tools_layout.addWidget(self.input_headline)
        
        tools_layout.addWidget(QLabel("<font color='white'>৩. অফার বা বিবরণ (Subheading):</font>"))
        self.input_subheading = QLineEdit("সীমিত সময়ের জন্য প্রিমিয়াম ওয়ালেট।")
        self.input_subheading.setStyleSheet(self.input_style())
        self.input_subheading.textChanged.connect(self.update_preview)
        tools_layout.addWidget(self.input_subheading)
        
        tools_layout.addWidget(QLabel("<font color='white'>৪. বাটনের টেক্সট (CTA):</font>"))
        self.input_cta = QLineEdit("Shop Now")
        self.input_cta.setStyleSheet(self.input_style())
        self.input_cta.textChanged.connect(self.update_preview)
        tools_layout.addWidget(self.input_cta)
        
        tools_layout.addStretch()
        
        self.btn_export = QPushButton("💾 ফাইনাল অ্যাড সেভ করুন (PNG)")
        self.btn_export.setFont(QFont("Arial", 11, QFont.Weight.Bold))
        self.btn_export.setStyleSheet(self.btn_style("#107c41"))
        self.btn_export.clicked.connect(self.export_ad)
        tools_layout.addWidget(self.btn_export)
        
        preview_frame = QFrame()
        preview_frame.setStyleSheet("background-color: #1e1e1e; border-radius: 8px;")
        preview_layout = QVBoxLayout(preview_frame)
        preview_layout.setAlignment(Qt.AlignmentFlag.AlignCenter)
        
        self.canvas_preview = QLabel()
        self.canvas_preview.setFixedSize(500, 500)
        self.canvas_preview.setStyleSheet("border: 2px dashed #555555;")
        preview_layout.addWidget(self.canvas_preview)
        
        h_layout.addWidget(tools_frame)
        h_layout.addWidget(preview_frame)
        
        self.update_preview()

    def btn_style(self, color):
        return f"background-color: {color}; color: white; border: none; padding: 10px; border-radius: 5px;"
    
    def input_style(self):
        return "background-color: #3c3c3c; color: white; border: 1px solid #555555; padding: 8px; border-radius: 4px;"

    def generate_ad_image(self, width=1080, height=1080):
        ad_img = Image.new("RGBA", (width, height), color=self.bg_color)
        draw = ImageDraw.Draw(ad_img)
        
        try:
            font_head = ImageFont.truetype("arial.ttf", 65)
            font_sub = ImageFont.truetype("arial.ttf", 42)
            font_cta = ImageFont.truetype("arial.ttf", 38)
        except IOError:
            font_head = font_sub = font_cta = ImageFont.load_default()

        draw.text((80, 250), self.input_headline.text(), fill=self.headline_color, font=font_head)
        draw.text((80, 400), self.input_subheading.text(), fill=self.subheading_color, font=font_sub)
        
        btn_coords = [(80, 700), (420, 790)]
        draw.rounded_rectangle(btn_coords, radius=15, fill="#e60000")
        draw.text((140, 720), self.input_cta.text(), fill="#ffffff", font=font_cta)
        
        return ad_img

    def update_preview(self):
        pil_img = self.generate_ad_image()
        pil_img_resized = pil_img.resize((500, 500))
        
        data = pil_img_resized.convert("RGBA").tobytes("raw", "RGBA")
        from PyQt6.QtGui import QImage, QPixmap
        qimg = QImage(data, 500, 500, QImage.Format.Format_RGBA8888)
        self.canvas_preview.setPixmap(QPixmap.fromImage(qimg))

    def select_bg_color(self):
        color = QColorDialog.getColor()
        if color.isValid():
            self.bg_color = color.name()
            self.update_preview()

    def export_ad(self):
        file_path, _ = QFileDialog.getSaveFileName(self, "Save Ad Banner", "My_Photoshop_Ad.png", "PNG Files (*.png)")
        if file_path:
            final_ad = self.generate_ad_image(1080, 1080)
            final_ad.convert("RGB").save(file_path, "PNG")

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = PhotoshopAdClone()
    window.show()
    sys.exit(app.exec())
