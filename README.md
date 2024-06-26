# uniapp-document-scanner

A uni-app demo of document scanner. It is mainly targeted for Weixin's mini program.

## Demo Video

* Scan documents via document scanners

    https://github.com/xulihang/uniapp-document-scanner/assets/5462205/a7fd0e4d-3dfa-4054-a245-8291aac23fbd

* Scan documents via the camera

    https://github.com/xulihang/uniapp-document-scanner/assets/5462205/c7a439d5-6b67-4901-ad5b-1e56bb7b7f21


## How it Works

### Document Scanner

It communicates with Dynamsoft Service to access document scanners.

Dynamsoft Service is provided by [Dynamic Web TWAIN](https://www.dynamsoft.com/web-twain/overview). It provides a REST API for acquiring images from scanners.

You need to set up two things to use it:

1. License for Dynamic Web TWAIN. Apply for one [here](https://www.dynamsoft.com/customer/license/trialLicense/?product=dwt).
2. Install Dynamsoft Service and make it available through an intranet so that the program can use it. [Guide](https://www.dynamsoft.com/codepool/flutter-twain-scanner-digitize-document.html#setting-up-dynamsoft-service)


### Camera

It runs Mobile Web Capture in a webview to scan documents via cameras and save the images as a PDF.

You can apply for its license [here](https://www.dynamsoft.com/customer/license/trialLicense/?product=mwc).



