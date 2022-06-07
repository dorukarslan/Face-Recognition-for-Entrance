# TEDUPASS
Face recognition system for campus entrance


![ezgif com-gif-maker(3)](https://user-images.githubusercontent.com/79598598/172455958-982e850d-e510-4606-bcbf-3442b3fc7a0a.gif)





### Technologies in TEDUPASS

In the TEDU PASS project, it was determined which technologies, frameworks, and libraries will be used to detect, recognize people's entrance and exit from buildings, and at the same time stay in live communication with the database. There are 4 technologies in the system. Arduino, python OpenCV, Mongo Database, FastApi. Since the system is still in the testing phase, Arduino connects the code to a system other than the computer and tests it is signaling with it. OpenCV performs the process of controlling the computer's camera, taking videos, and creating frames around the recognized faces. Mongo Database is used to keep the times when students log in and out and send them to the front when necessary. When the data created in the FastApi database is needed by both the backside and the front side, this information is pulled from the database and used to implement it into the code. After the technologies were determined, it came to the design stage of the system. As a result of various literature reviews, it was decided to convert a photo into a 128-dimensional array using the encoding method, which is one of the most efficient methods for face capture and recognition, and compare it with faces captured live with capturing video. After finding an efficient face recognition and capture method, it was thought about how the system elements would work. In this system designed for the school, the average time spent by Ted university students reading their cards and leaving school was calculated and it was determined that this time was 6 seconds for entry or exit. Because of this, the system is designed so that someone can enter or exit every 6 seconds. In addition, because the video capture and face recognition system has been designed to capture multiple faces at the same time, the possibility of students experiencing difficulties in the process of entering or leaving the school has arisen. To correct this problem, using the frame structure in image processing, it was ensured that only the faces of people who are at a certain proximity to the video camera were entered into the matching process. A semi-local system for database communication was designed at the final stage since it is quite costly for image processing areas to communicate with a real-time database. As a result, thanks to the literature review and proper system design, a project has emerged that can work as a prototype with a certain accuracy and efficiency ratio.


# Face Detection and Recognize 

At the initial stage of face recognition and detection, a video capture system was installed as a priority. The Video Capture() function of OpenCV was used to perform the video capture process. This is the function to start the camera's power-on and image acquisition. After the video capture process is finished, the next step is to capture the faces in the video. For this purpose, the video frame is resized by 1/4, although it reduces the accuracy rate, primarily to capture and identify faces faster. Then, the images captured in the video are converted from BGR format to RGB format by reversing so that the face_recognition library can use it.
After the captured face images are properly formatted in the face_recognition library, the face recognition process begins. First of all, the location of the face captured on the video is found in real-time. Then the face image that is defined with location is converted into a 128dimensional array using the encoding() function in the face_recognition library. After all these stages, the encoded image is again uploaded to the database by students and faculty members using compare_face(), which is a function of the same library and compared with the images stored as a 128-dimensional array. As a result of comparing the sequences, the images closest to the sequence in the video capture are detected and kept in a list. When comparing arrays, they are compared according to a certain similarity ratio,  and this value is sent as
a tolerance value in the function parameter. According to the research, the similarity ratio of 0.6 as the tolerance value was determined as the most efficient value and was applied in the project.
The lower the specified tolerance value of a 128-dimensional array, the greater the similarity to the image captured on the video. To calculate this similarity ratio, the face_distance() function, which is one of the functions of the face_recognition() library, is used. This function compares the images in the database with the image captured from the video and calculates a euclidean distance for each comparison. Then, by applying the argmin() function of the NumPy library to these calculated euclidean distances, the minimum euclidean distance, that is, the measure of the image closest to the image in the video recording, is determined. If this measurement belongs to one of the images that previously matched the image in the video from the database and is stored in the list, the face is detected as defined. When the system detects that this student/staff is familiar, it creates a red square frame around his/her face. Under this frame, the name and surname of that person registered to the system are created as a separate frame. This frame created around the face grows and shrinks in proportion to the size of the person's face and the degree of proximity to the camera. If the faces found here are also registered in the system, they are added to the face_names array in the locale. Also, The system treats unrecognized faces as unknown and exempts them from many operations. An unknown face system and recording it locally in a given dynamic way benefit the system in terms of speed and security. After the face detection, recognition, and frame creation processes for the face, the process of communicating with the database begins.


## Database Communication
The database communication starts by taking the faces and surnames of the students and staff registered in the system to the locale. This information is in the Mongo Database before this process and is performed with the find () function of FastApi.
There are 3 basic conditions for face detection and sending to the system. Your face must be within the camera's set proximity. Its name should be the Unknown (that is, it should be registered to the system) flag, and it is used to distinguish whether the detection made by the camera at that moment is for input or output. If this value is 0, it means that the student/staff is not at the school, if it is 1, it is inside the school. If these 3 conditions are met, the time that this person appears on the camera should be recorded in the database. In other words, it should be determined what time he came to school and what time he left. These data should be sent to both the front side and the database for the security system. In addition, after a student/staff's face is recognized, this face is not recognized for about 6 seconds. This is used to prevent the entrant from logging in more than once. The system is waiting for this person to come out from the camera’s point of view. If such a precaution were not taken, the same person would have been deemed to have logged in and out during his stay at the camera angle. To solve this problem, 2 separate input and output cameras could be used, but the single- camera system was preferred both to reduce the cost and to make the system safe. Within this expected 6 seconds, other staff/students can continue to enter and exit the school. This state of the process is personal only. In other words, there is no slowdown or disruption in the system. After this 6-second wait, if this person enters the camera again, the system now perceives that person as logging out. These 6 seconds can be changed according to the user and demand. While these operations are saved in the database, they are also saved in the locale. This increases the entry-exit times and security of the process.

## User Interface

In the Tedu Pass project, an interface has been developed for tracking logs and installing new users in the system. When designing this interface, a simple appearance was created with the help of HTML and CSS. To translate this web design info into a web application, the FastApi framework was used. The design was visualized in the browser with the Jinja2 templates function of Fast Api. An endpoint was created to add users, and the information received from the form in the interface was sent to the database. However, before this operation was performed, the user's photo taken from the form was brought to the format requested by the face_recognize library and sent to the database.


##  Non-Relational Database
In the TEDU PASS project, the MongoDB database, which is a
nonrelational database, was preferred for speed and efficiency because the code is in constant communication with the camera and the database. There are two collections in the selected non-relational database. These are the event_log and users collections In the collection of users, the names, surnames, and facial recognition of all students and faculty members whose school has been registered and whose photos have been registered in the system using the interface of the system are kept in the 128-dimensional array version of the photo for use in face recognition. In the event_log collection, if the face of one of the users registered in the system is detected at the entrance or exit of the school, this collection is registered as a log, and only in the security-specific interface, these records are displayed live. In this way, when and how students and faculty members enter the school are recorded in the system.
