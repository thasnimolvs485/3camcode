# 3camcode
using basler camera
###############################################################################
# Imports
###############################################################################
from pypylon import pylon
from pypylon import genicam
from datetime import datetime

import time

# Import OpenCV
import cv2
from PIL import Image

# Import os for creating directories.
import os

#os.environ["PYLON_CAMEMU"] = "2"
import sys
# Import shutil for deleting folders.
import shutil

# Import threading for creating threads.
import threading

import numpy as np



###############################################################################
# Parameters
###############################################################################
#countOfImagesToGrab=60
#fps=60
#outputDir = '/mnt/ramdisk/output'
#outputDir = '/mnt/Disk3/test/output'
outputDir = '/mnt/Disk3/test/trial'
#outputDir = '/mnt/Dicccsk3/test/trial'

#outputDir = 'output'

###############################################################################
# Global variables
###############################################################################
  # Get the transport layer factory.
tlFactory = pylon.TlFactory.GetInstance()
# Get all attached devices and exit application if no device is found.
devices = tlFactory.EnumerateDevices()
print(devices)

class CameraImage:
    
    def __init__(self, device):
        self.runningStatus = False
        self.thread = None
        self.imageList = []
        self.device = device
        
    def frameGrabber(self):
        cam = pylon.InstantCamera(tlFactory.CreateDevice(self.device))
        
        cam.Open()
        
        cam.Width= 1440
        cam.Height=1080
        cam.ExposureTime = 3000
        #cam.TriggerMode.SetValue("On")
        cam.TriggerSelector.SetValue("FrameBurstStart")
        cam.TriggerMode.SetValue("On") #hardware trigger
        cam.TriggerSource.SetValue("Line4")
        cam.TriggerActivation.SetValue('RisingEdge')
        #cam.TriggerSelector.SetValue("FrameStart") # FrameBurstStart is used for capturing a series of images with a single signal, please choose FrameStart if wish to have one signal one image
        #cam.TriggerDelay.SetValue(300)
        
        #cam.TriggerSource.SetValue("Line4")
        
        
        #cam.TriggerActivation.SetValue('RisingEdge')
        cam.PgiMode.SetValue("On")
        cam.NoiseReduction.SetValue(2.0)
        cam.SharpnessEnhancement.SetValue(3.984365)
       # cam.MaxNumBuffer=1000
       # // Select the frame start trigger
      #  cam.TriggerSelector.SetValue(TriggerSelector_FrameStart);
       # // Set the delay for the frame start trigger to 300 µs
       #camera.Gain.SetValue(22.5)
        #camera.Gain.SetValue(23.5) 
       # cam.SharpnessEnhancement.SetValue(3.0)
        #cam.AcquisitionFrameRateEnable.SetValue(True)
        #cam.AcquisitionFrameRate.SetValue(200.0) 
        
       
    # Grab c_countOfImagesToGrab from the cameras.
        startTime = time.time()
#x_max=200
#x_current=0
        #cam.StartGrabbingMax()
        cam.StartGrabbing()
#while   x_current< x_max:
        
        while cam.IsGrabbing():
            grabResult = cam.RetrieveResult(5000, pylon.TimeoutHandling_ThrowException)
            
            # Image grabbed successfully?
            if grabResult.GrabSucceeded():
                serial_number = '_'.join([cam.GetDeviceInfo().GetModelName(),
                            cam.GetDeviceInfo().GetSerialNumber()])
                #print('Serial No:%s' %serial_number) 
                imageDict = {}
                imageDict['filename'] = datetime.now().strftime("%Y%m%d-%H%M%S%f-{}.tiff".format(serial_number))
                imageDict['image'] = grabResult.GetArray()
                self.runningStatus = True
            # Push the frame to the queue.
                self.imageList.append(imageDict)
                 
            else:
               #self.runningStatus=False 
               print("Error: ", grabResult.ErrorCode, grabResult.ErrorDescription)
            grabResult.Release()
            
        #x_current= time.time()-startTime

        self.runningStatus = False
        #endTime = time.time()
        #print('Total time = %f' % (endTime - startTime))

    def startGrabbing(self):
        self.thread = threading.Thread(target=self.frameGrabber)
        self.thread.start()
    
###############################################################################
# Class to save video frames.

class SaveImage:

    def __init__(self, cameraImage):
        self.cameraImage = cameraImage
        self.thread = None
    
    def imageSaver(self):
        totalTime = 0
        while not self.cameraImage.runningStatus:
            time.sleep(0.001)
        while True:
            if not self.cameraImage.runningStatus and len(self.cameraImage.imageList) == 0:
                break
            if len(self.cameraImage.imageList) == 0:
                continue
            else:
                imageDict = self.cameraImage.imageList.pop(0)
                img = imageDict['image'] 
                #print('img')
                filename = imageDict['filename']
                #print('filename')
                #startTime = time.time()
                #cv2.imwrite('%s/%s' % (outputDir, filename), img)
                im = Image.fromarray(img)
                #print('im')
                im.save('%s/%s' % (outputDir, filename))
                totalTime = totalTime + (time.time() - startTime)
        print('Total time for saving: %f' % totalTime)
                 
    def startSaving(self):
        self.thread = threading.Thread(target=self.imageSaver)
        self.thread.start()

    
###############################################################################
# Main code
###############################################################################

# Reset the output directory.
if os.path.isdir(outputDir):
    shutil.rmtree(outputDir)
os.mkdir(outputDir)
i=3
while True:
    #print (i)
    startTime = time.time()
        # Create the video object.
    cameraImage1 = CameraImage(devices[0])
    saveImage1 = SaveImage(cameraImage1)

    cameraImage2 = CameraImage(devices[1])
    saveImage2 = SaveImage(cameraImage2)

    cameraImage3 = CameraImage(devices[2])
    saveImage3 = SaveImage(cameraImage3)


    startTime = time.time()
    cameraImage1.startGrabbing()
    saveImage1.startSaving()
    cameraImage2.startGrabbing()
    saveImage2.startSaving()
    cameraImage3.startGrabbing()
    saveImage3.startSaving()

    cameraImage1.thread.join()
    saveImage1.thread.join()
    cameraImage2.thread.join()
    saveImage2.thread.join()
    cameraImage3.thread.join()
    saveImage3.thread.join()


    # End timing.
    endTime = time.time()

    # Print the time taken.
    print("Total time taken = %f seconds." % (endTime - startTime))
    #print("stop")
    if i==0:

        break
        ###############################################################################
# End of file.
###############################################################################
