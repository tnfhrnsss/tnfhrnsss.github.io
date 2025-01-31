---
layout: post
title: PngImageFile object has no attribute load_seek
date: 2025-02-01 00:00:00
last_modified_at : 2025-02-01 00:00:00
parent: Python
has_children: false
nav_order: 1
nav_exclude: true
---

# Error

```
(venv) ~/Documents/_P/eumang/src/main git:[main]
python3 ocr_.py
./img/phishing_3.jpg
---------
phishing_3.jpg
#########
Traceback (most recent call last):
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/
  PIL/ImageFile.py", line 204, in load
    seek = self.load_seek
           ^^^^^^^^^^^^^^
AttributeError: 'PngImageFile' object has no attribute 'load_seek'. Did you mean: 'load_end'?

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/Documents/_P/eumang/src/main/ocr_.py", line 34, in <module>
    extract_text_from_images()
  File "/Documents/_P/eumang/src/main/ocr_.py", line 23, in extract_text_from_images
    text = pytesseract.image_to_string(image, lang='kor')
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/pytesseract/pytesseract.py", line 423, in image_to_string
    return {
           ^
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/pytesseract/pytesseract.py", line 426, in <lambda>
    Output.STRING: lambda: run_and_get_output(*args),
                           ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/pytesseract/pytesseract.py", line 277, in run_and_get_output
    with save(image) as (temp_name, input_filename):
  File "/usr/local/Cellar/python@3.12/3.12.4/Frameworks/Python.framework/Versions/3.12/lib/python3.12/contextlib.py", line 137, in __enter__
    return next(self.gen)
           ^^^^^^^^^^^^^^
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/pytesseract/pytesseract.py", line 199, in save
    image.save(input_file_name, format=image.format)
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/PIL/Image.py", line 2528, in save
    self._ensure_mutable()
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/PIL/Image.py", line 639, in _ensure_mutable
    self._copy()
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/PIL/Image.py", line 632, in _copy
    self.load()
  File "/Documents/_P/eumang/src/main/path/to/venv/lib/python3.12/site-packages/PIL/ImageFile.py", line 207, in load
    seek = self.fp.seek
           ^^^^^^^^^^^^
AttributeError: 'NoneType' object has no attribute 'seek'
```

# Try..try-catch

- try-catch로 감싸고 나니 image 오류가 아닌 **“tesseract is not installed or it's not in your PATH.”** 에러가 나왓다.
- 찾다보니 tesseract 말고 tesseract-ocr가 필요했던 것으로, 다시 설치하고 실행하니 아래 오류 발생

```prolog
Building wheel for tesseract-ocr (pyproject.toml) ... error
error: subprocess-exited-with-error
```

- [https://forum.robotframework.org/t/unable-to-install-tesseract-ocr-through-pip-command/6605](https://forum.robotframework.org/t/unable-to-install-tesseract-ocr-through-pip-command/6605){:target="_blank"} 참고

# Solved

- 결국엔 brew install tesseract , brew install tesseract-ocr, brew install tesseract-lang 모두 설치했다
- brew로 python 설치했더니….늘 까먹는다