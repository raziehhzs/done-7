#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
اسکریپت سامان‌دهندهٔ هوشمند فایل‌ها
------------------------------------
این ابزار یک اسکریپت تک‌فایلی پایتون است که پوشهٔ انتخابی شما (مثلاً Downloads)
را بر اساس «نوع فایل» یا «تاریخ» منظم می‌کند، از برخورد نام‌ها جلوگیری
می‌کند، لاگ کامل جابجایی‌ها را ذخیره می‌کند و امکان «برگشت/Undo» را فراهم می‌کند.

ویژگی‌ها:
- دو حالت گروه‌بندی: بر اساس نوع فایل (Documents/Images/...) یا بر اساس تاریخ (YYYY/MM)
- Dry-run برای پیش‌نمایش تغییرات بدون اعمال واقعی
- مدیریت برخورد نام (افزودن شمارنده یا هش کوتاه)
- نادیده‌گرفتن الگوها/پوشه‌ها‌ی خاص
- قابلیت Undo با استفاده از لاگ JSON زمان‌مند
- فیلتر بر اساس حداقل/حداکثر اندازه و بازهٔ زمانی آخرین تغییر

نحوهٔ استفادهٔ سریع:
    python organizer.py ~/Downloads --mode type --dry-run
    python organizer.py ~/Downloads --mode date
    python organizer.py ~/Downloads --mode type --undo آخرین-لاگ-ثبت‌شده

برای جزئیات کامل:
    python organizer.py -h
"""

from __future__ import annotations
import argparse
import dataclasses as dc
import fnmatch
import hashlib
import json
import os
from pathlib import Path
import shutil
import sys
from datetime import datetime, timezone
from typing import Dict, Iterable, List, Optional, Tuple

# --- پیکربندی پیش‌فرض ---
DEFAULT_TYPE_BUCKETS = {
    "Images": ["*.jpg", "*.jpeg", "*.png", "*.gif", "*.bmp", "*.tiff", "*.webp", "*.heic"],
    "Videos": ["*.mp4", "*.mov", "*.mkv", "*.avi", "*.wmv", "*.webm", "*.m4v"],
    "Audio": ["*.mp3", "*.wav", "*.flac", "*.aac", "*.ogg", "*.m4a"],
    "Documents": ["*.pdf", "*.doc", "*.docx", "*.xls", "*.xlsx", "*.ppt", "*.pptx", "*.txt", "*.md", "*.rtf", "*.odt"],
    "Archives": ["*.zip", "*.rar",

