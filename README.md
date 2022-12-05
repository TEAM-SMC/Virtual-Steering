# Virtual-Steering
We will be having 3 files:
-	keyinput.py
-	requirements.txt
-	steering.py

1.	requirements.txt 
We will be installing 2 packages using pip, namely OpenCV and Mediapipe. Pip is a recommended package-management system written in Python and is used to install and manage software packages. It connects to an online repository of public packages, called the Python Package Index. A package contains all the files you need for a module. Modules are Python code libraries you can include in your project. 
If you do not have a latest version of pip type “pip install --upgrade pip” in the terminal.
To check various pip commands type “pip --help” in the terminal.

OpenCV: OpenCV (Open-Source Computer Vision Library) is an open-source computer vision and machine learning software library. OpenCV was built to provide a common infrastructure for computer vision applications and to accelerate the use of machine perception in the commercial products.
To install OpenCV type “pip install opencv-python” in the terminal.

Mediapipe:  MediaPipe is a cross-platform pipeline (sequence of data processing mechanisms) framework to build custom machine learning solutions for live and streaming media. The framework was open-sourced by Google hat provides amazing ready-to-use ML solutions for computer vision tasks.
To install Mediapipe type “pip install mediapipe” in the terminal.



2.	Keyinput.py
ctypes: ctypes is a foreign function library for Python. It provides C compatible data types, and allows calling functions in DLLs or shared libraries. It can be used to wrap these libraries in pure Python.
```
import ctypes

keys = {
    "w":0x11,
    "a":0x1E,
    "s":0x1F,
    "d":0x20,
}
```
For keyboard scan codes go [here](https://web.archive.org/web/20190801085838/http:/www.gamespp.com/directx/directInputKeyboardScanCodes.html)
-	ctypes.POINTER(type)
This factory function creates and returns a new ctypes pointer type. Pointer types are cached and reused internally, so calling this function repeatedly is cheap. type must be a ctypes type.
```
PUL = ctypes.POINTER(ctypes.c_ulong)
class KeyBdInput(ctypes.Structure):
    _fields_ = [("wVk", ctypes.c_ushort),
                ("wScan", ctypes.c_ushort),
                ("dwFlags", ctypes.c_ulong),
                ("time", ctypes.c_ulong),
                ("dwExtraInfo", PUL)]

class HardwareInput(ctypes.Structure):
    _fields_ = [("uMsg", ctypes.c_ulong),
                ("wParamL", ctypes.c_short),
                ("wParamH", ctypes.c_ushort)]

class MouseInput(ctypes.Structure):
    _fields_ = [("dx", ctypes.c_long),
                ("dy", ctypes.c_long),
                ("mouseData", ctypes.c_ulong),
                ("dwFlags", ctypes.c_ulong),
                ("time",ctypes.c_ulong),
                ("dwExtraInfo", PUL)]

class Input_I(ctypes.Union):
    _fields_ = [("ki", KeyBdInput),
                 ("mi", MouseInput),
                 ("hi", HardwareInput)]

class Input(ctypes.Structure):
    _fields_ = [("type", ctypes.c_ulong),
                ("ii", Input_I)]
```
-	class ctypes.Structure(*args, **kw)¶
Abstract base class for structures in native byte order.
Concrete structure and union types must be created by subclassing one of these types, and at least define a _fields_ class variable. ctypes will create descriptors which allow reading and writing the fields by direct attribute accesses. 
-	class ctypes.c_long
Represents the C signed long datatype. The constructor accepts an optional integer initializer; no overflow checking is done.
-	class ctypes.c_ulong
Represents the C unsigned long datatype. The constructor accepts an optional integer initializer; no overflow checking is done.
-	class ctypes.c_ushort
Represents the C unsigned short datatype. The constructor accepts an optional integer initializer; no overflow checking is done.
-	class ctypes.Union(*args, **kw)
Abstract base class for unions in native byte order.
-	_fields_
A sequence defining the structure fields. The items must be 2-tuples or 3-tuples. The first item is the name of the field, the second item specifies the type of the field; it can be any ctypes data type.
For integer type fields like c_int, a third optional item can be given. It must be a small positive integer defining the bit width of the field. Field names must be unique within one structure or union. This is not checked, only one field can be accessed when names are repeated.
It is possible to define the _fields_ class variable after the class statement that defines the Structure subclass, this allows creating data types that directly or indirectly reference themselves:
class List(Structure):
    pass
List._fields_ = [("pnext", POINTER(List)),
                 ...
                ]
The _fields_ class variable must, however, be defined before the type is first used (an instance is created, sizeof() is called on it, and so on). Later assignments to the _fields_ class variable will raise an AttributeError.
It is possible to define sub-subclasses of structure types, they inherit the fields of the base class plus the _fields_ defined in the sub-subclass, if any.
# Actual Functions
```
def press_key(key):
    extra = ctypes.c_ulong(0)
    ii_ = Input_I()
    ii_.ki = KeyBdInput( 0, keys[key], 0x0008, 0, ctypes.pointer(extra) )
    x = Input( ctypes.c_ulong(1), ii_ )
    ctypes.windll.user32.SendInput(1, ctypes.pointer(x), ctypes.sizeof(x))

def release_key(key):
    extra = ctypes.c_ulong(0)
    ii_ = Input_I()
    ii_.ki = KeyBdInput( 0, keys[key], 0x0008 | 0x0002, 0, ctypes.pointer(extra) )
    x = Input( ctypes.c_ulong(1), ii_ )
    ctypes.windll.user32.SendInput(1, ctypes.pointer(x), ctypes.sizeof(x))
```
-	ctypes.pointer(obj)
This function creates a new pointer instance, pointing to obj. The returned object is of the type POINTER(type(obj)).
Note: If you just want to pass a pointer to an object to a foreign function call, you should use byref(obj) which is much faster.
-	class ctypes.WinDLL(name, mode=DEFAULT_MODE, handle=None, use_errno=False, use_last_error=False, winmode=None)¶
Windows only: Instances of this class represent loaded shared libraries, functions in these libraries use the stdcall calling convention, and are assumed to return int by default.
The Python global interpreter lock is released before calling any function exported by these libraries, and reacquired afterwards.

3.	steering.py
```
import math # for mathematical funvtions
import keyinput # for key inputs defined in the other file
import cv2
import mediapipe as mp
mp_drawing = mp.solutions.drawing_utils # Initializing the drawng utilities for drawing landmarks on image
mp_drawing_styles = mp.solutions.drawing_styles # Initializing the drawng styles for drawing landmarks on image
mp_hands = mp.solutions.hands # initialize the Hands class
font = cv2.FONT_HERSHEY_SIMPLEX #font style
# 0 For webcam input:
cap = cv2.VideoCapture(0)
# using mp.hands to detect hands
with mp_hands.Hands(
    model_complexity=0,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5) as hands:
  while cap.isOpened():
    success, image = cap.read()
    if not success:
      print("Ignoring empty camera frame.")
      # If loading a video, use 'break' instead of 'continue'.
      continue

    # To improve performance, optionally mark the image as not writeable to
    image.flags.writeable = False
    image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    results = hands.process(image)
    imageHeight, imageWidth, _ = image.shape
```
-	MODEL_COMPLEXITY
Complexity of the hand landmark model: 0 or 1. Landmark accuracy as well as inference latency generally go up with the model complexity. Default to 1.
-	MIN_DETECTION_CONFIDENCE
Minimum confidence value ([0.0, 1.0]) from the hand detection model for the detection to be considered successful. Default to 0.5.
-	MIN_TRACKING_CONFIDENCE:
Minimum confidence value ([0.0, 1.0]) from the landmark-tracking model for the hand landmarks to be considered tracked successfully, or otherwise hand detection will be invoked automatically on the next input image. Setting it to a higher value can increase robustness of the solution, at the expense of a higher latency. Ignored if static_image_mode is true, where hand detection simply runs on every image. Default to 0.5.
# Draw the hand annotations on the image.
    image.flags.writeable = True
    image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
    co=[] #list
    if results.multi_hand_landmarks:
      for hand_landmarks in results.multi_hand_landmarks:
        mp_drawing.draw_landmarks(
            image,
            hand_landmarks,
            mp_hands.HAND_CONNECTIONS,
            mp_drawing_styles.get_default_hand_landmarks_style(),
            mp_drawing_styles.get_default_hand_connections_style())
        for point in mp_hands.HandLandmark:
           if str(point) == "HandLandmark.WRIST":
              normalizedLandmark = hand_landmarks.landmark[point]
              pixelCoordinatesLandmark = mp_drawing._normalized_to_pixel_coordinates(normalizedLandmark.x,
                                                                                        normalizedLandmark.y,
                                                                                    imageWidth, imageHeight)

              try:
                co.append(list(pixelCoordinatesLandmark))
              except:
                  continue

-	Hand Landmark Model
After the palm detection over the whole image our subsequent hand landmark model performs precise keypoint localization of 21 3D hand-knuckle coordinates inside the detected hand regions via regression, that is direct coordinate prediction. The model learns a consistent internal hand pose representation and is robust even to partially visible hands and self-occlusions.
To obtain ground truth data, we have manually annotated ~30K real-world images with 21 3D coordinates, as shown below (we take Z-value from image depth map, if it exists per corresponding coordinate). To better cover the possible hand poses and provide additional supervision on the nature of hand geometry, we also render a high-quality synthetic hand model over various backgrounds and map it to the corresponding 3D coordinates
 
-	MULTI_HAND_LANDMARKS
Collection of detected/tracked hands, where each hand is represented as a list of 21 hand landmarks and each landmark is composed of x, y and z. x and y are normalized to [0.0, 1.0] by the image width and height respectively. z represents the landmark depth with the depth at the wrist being the origin, and the smaller the value the closer the landmark is to the camera. The magnitude of z uses roughly the same scale as x.
# mapping of points
    if len(co) == 2:
        xm, ym = (co[0][0] + co[1][0]) / 2, (co[0][1] + co[1][1]) / 2
        radius = 150
        try:
            m=(co[1][1]-co[0][1])/(co[1][0]-co[0][0])
        except:
            continue
        a = 1 + m ** 2
        b = -2 * xm - 2 * co[0][0] * (m ** 2) + 2 * m * co[0][1] - 2 * m * ym
        c = xm ** 2 + (m ** 2) * (co[0][0] ** 2) + co[0][1] ** 2 + ym ** 2 - 2 * co[0][1] * ym - 2 * co[0][1] * co[0][
            0] * m + 2 * m * ym * co[0][0] - 22500
        xa = (-b + (b ** 2 - 4 * a * c) ** 0.5) / (2 * a)
        xb = (-b - (b ** 2 - 4 * a * c) ** 0.5) / (2 * a)
        ya = m * (xa - co[0][0]) + co[0][1]
        yb = m * (xb - co[0][0]) + co[0][1]
        if m!=0:
          ap = 1 + ((-1/m) ** 2)
          bp = -2 * xm - 2 * xm * ((-1/m) ** 2) + 2 * (-1/m) * ym - 2 * (-1/m) * ym
          cp = xm ** 2 + ((-1/m) ** 2) * (xm ** 2) + ym ** 2 + ym ** 2 - 2 * ym * ym - 2 * ym * xm * (-1/m) + 2 * (-1/m) * ym * xm - 22500
          try:
           xap = (-bp + (bp ** 2 - 4 * ap * cp) ** 0.5) / (2 * ap)
           xbp = (-bp - (bp ** 2 - 4 * ap * cp) ** 0.5) / (2 * ap)
           yap = (-1 / m) * (xap - xm) + ym
           ybp = (-1 / m) * (xbp - xm) + ym

          except:
              continue

Try and Except statement is used to handle these errors within our code in Python. The try block is used to check some code for errors i.e., the code inside the try block will execute when there is no error in the program. Whereas the code inside the except block will execute whenever the program encounters some error in the preceding try block.
How try() works? 
First, the try clause is executed i.e., the code between try and except clause. If there is no exception, then only the try clause will run, except the clause is finished. If any exception occurs, the try clause will be skipped and except clause will run. If any exception occurs, but the except clause within the code doesn’t handle it, it is passed on to the outer try statements. If the exception is left unhandled, then the execution stops. A try statement can have more than one except clause.
Python continue statement is one of the loop statements that control the flow of the loop. More specifically, the continue statement skips the “rest of the loop” and jumps into the beginning of the next iteration.
Unlike the break statement, the continue does not exit the loop.
cv2.circle(img=image,center=(int(xm),int(ym)),radius=radius,color=(195, 255, 62), thickness=15)

        l = (int(math.sqrt((co[0][0] - co[1][0]) ** 2 * (co[0][1] - co[1][1]) ** 2)) - 150) // 2
        cv2.line(image, (int(xa), int(ya)), (int(xb), int(yb)), (195, 255, 62), 20)
        if co[0][0] > co[1][0] and co[0][1]>co[1][1] and co[0][1] - co[1][1] > 65:
            print("Turn left.")
            keyinput.release_key('s')
            keyinput.release_key('d')
            keyinput.press_key('a')
            cv2.putText(image, "Turn left", (50, 50), font, 0.8, (0, 255, 0), 2, cv2.LINE_AA)
            cv2.line(image, (int(xbp), int(ybp)), (int(xm), int(ym)), (195, 255, 62), 20)

        elif co[1][0] > co[0][0] and co[1][1]> co[0][1] and co[1][1] - co[0][1] > 65:
            print("Turn left.")
            keyinput.release_key('s')
            keyinput.release_key('d')
            keyinput.press_key('a')
            cv2.putText(image, "Turn left", (50, 50), font, 0.8, (0, 255, 0), 2, cv2.LINE_AA)
            cv2.line(image, (int(xbp), int(ybp)), (int(xm), int(ym)), (195, 255, 62), 20)

        elif co[0][0] > co[1][0] and co[1][1]> co[0][1] and co[1][1] - co[0][1] > 65:
            print("Turn right.")
            keyinput.release_key('s')
            keyinput.release_key('a')
            keyinput.press_key('d')
            cv2.putText(image, "Turn right", (50, 50), font, 0.8, (0, 255, 0), 2, cv2.LINE_AA)
            cv2.line(image, (int(xap), int(yap)), (int(xm), int(ym)), (195, 255, 62), 20)

        elif co[1][0] > co[0][0] and co[0][1]> co[1][1] and co[0][1] - co[1][1] > 65:
            print("Turn right.")
            keyinput.release_key('s')
            keyinput.release_key('a')
            keyinput.press_key('d')
            cv2.putText(image, "Turn right", (50, 50), font, 0.8, (0, 255, 0), 2, cv2.LINE_AA)
            cv2.line(image, (int(xap), int(yap)), (int(xm), int(ym)), (195, 255, 62), 20)
        
        else:
            print("keeping straight")
            keyinput.release_key('s')
            keyinput.release_key('a')
            keyinput.release_key('d')
            keyinput.press_key('w')
            cv2.putText(image, "keep straight", (50, 50), font, 0.8, (0, 255, 0), 2, cv2.LINE_AA)
            if ybp>yap:
                cv2.line(image, (int(xbp), int(ybp)), (int(xm), int(ym)), (195, 255, 62), 20)
            else:
                cv2.line(image, (int(xap), int(yap)), (int(xm), int(ym)), (195, 255, 62), 20)

    if len(co)==1:
       print("keeping back")
       keyinput.release_key('a')
       keyinput.release_key('d')
       keyinput.release_key('w')
       keyinput.press_key('s')
       cv2.putText(image, "keeping back", (50, 50), font, 1.0, (0, 255, 0), 2, cv2.LINE_AA)

    cv2.imshow('MediaPipe Hands', cv2.flip(image, 1))
```
# Flip the image horizontally for a selfie-view display.
    if cv2.waitKey(5) & 0xFF == ord('q'):
      break
cap.release()
```
cv2.circle() method is used to draw a circle on any image.
Parameters: 
•	image: It is the image on which the circle is to be drawn. 
•	center_coordinates: It is the center coordinates of the circle. The coordinates are represented as tuples of two values i.e. (X coordinate value, Y coordinate value). 
•	radius: It is the radius of the circle. 
•	color: It is the color of the borderline of a circle to be drawn. For BGR, we pass a tuple. eg: (255, 0, 0) for blue color. 
•	thickness: It is the thickness of the circle border line in px. Thickness of -1 px will fill the circle shape by the specified color.
Return Value: It returns an image. 
The steps to create a circle on an image are:
1.	Read the image using imread() function.
2.	Pass this image to the cv2.circle() method along with other parameters such as center_coordinates, radius, color and thickness.
3.	Display the image using cv2.imshow() method.

cv2.line() method is used to draw a line on any image.
Parameters: image: It is the image on which line is to be drawn. 
•	start_point: It is the starting coordinates of the line. The coordinates are represented as tuples of two values i.e. (X coordinate value, Y coordinate value). 
•	end_point: It is the ending coordinates of the line. The coordinates are represented as tuples of two values i.e. (X coordinate value, Y coordinate value). 
•	color: It is the color of the line to be drawn. For RGB, we pass a tuple. eg: (255, 0, 0) for blue color.
•	thickness: It is the thickness of the line in px. 
Return Value: It returns an image.

cv2.putText() method is used to draw a text string on any image.
Parameters:
image: It is the image on which text is to be drawn.
text: Text string to be drawn.
org: It is the coordinates of the bottom-left corner of the text string in the image. The coordinates are represented as tuples of two values i.e. (X coordinate value, Y coordinate value).
font: It denotes the font type. Some of font types are FONT_HERSHEY_SIMPLEX, FONT_HERSHEY_PLAIN, , etc.
fontScale: Font scale factor that is multiplied by the font-specific base size.
color: It is the color of text string to be drawn. For BGR, we pass a tuple. eg: (255, 0, 0) for blue color.
thickness: It is the thickness of the line in px.
lineType: This is an optional parameter.It gives the type of the line to be used.
bottomLeftOrigin: This is an optional parameter. When it is true, the image data origin is at the bottom-left corner. Otherwise, it is at the top-left corner.

Return Value: It returns an image.
