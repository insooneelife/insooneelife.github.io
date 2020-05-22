---
title: "[Python] Opencv Template Match"
categories:
  - OpenCV
---

#### Template Match

템플릿 매칭(template matching)은 참조 영상(reference image)에서 템플릿(template) 영상과 매칭되는 위치를 탐색하는 방법이다. 일반적으로 템플릿 매칭은 이동(translation) 문제는 해결할 수 있는 반면, 회전 및 스케일링된 물체의 매칭은 어려운 문제이다. 템플릿 매칭에서 영상의 밝기를 그대로 사용할 수도 있고, 에지, 코너점, 주파수 변환 등의 특징 공간으로 변환하여 템플릿 매칭을 수행할 수 있으며, 영상의 밝기 등에 덜 민감하도록 정규화 과정이 필요하다.

다음은 파이썬 템플릿 매치 알고리즘 예제이다.

```python

import cv2 as cv
import numpy as np
from matplotlib import pyplot as plt

methods = ['cv.TM_CCOEFF', 'cv.TM_CCOEFF_NORMED', 'cv.TM_CCORR',
           'cv.TM_CCORR_NORMED', 'cv.TM_SQDIFF', 'cv.TM_SQDIFF_NORMED']

def template_match(img, template):
    w, h = template.shape[::-1]
    method = cv.TM_CCOEFF_NORMED

    # Apply template Matching
    res = cv.matchTemplate(img, template, method)
    min_val, max_val, min_loc, max_loc = cv.minMaxLoc(res)

    # If the method is TM_SQDIFF or TM_SQDIFF_NORMED, take minimum
    if method in [cv.TM_SQDIFF, cv.TM_SQDIFF_NORMED]:
        top_left = min_loc
    else:
        top_left = max_loc

    bottom_right = (top_left[0] + w, top_left[1] + h)
    cv.rectangle(img, top_left, bottom_right, 255, 2)

    print ("max val : ", max_val)

    plt.subplot(121), plt.imshow(res, cmap='gray')
    plt.title('Matching Result'), plt.xticks([]), plt.yticks([])
    plt.subplot(122), plt.imshow(img, cmap='gray')
    plt.title('Detected Point'), plt.xticks([]), plt.yticks([])
    plt.show()

capture = cv.VideoCapture("window.mp4")

prev_frame = None
for i in range(2):
    ret, frame = capture.read()
    cv.imshow("VideoFrame", frame)

    frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)

    if i == 1:
        template_match(frame, prev_frame)

    prev_frame = frame

    if cv.waitKey(33) > 0: break

cv.waitKey()
capture.release()
cv.destroyAllWindows()

```
