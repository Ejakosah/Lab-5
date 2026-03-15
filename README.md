# Lab-5
Task 1

![Screenshot](Screenshot1a.png)


Task 2
https://github.com/Ejakosah/Lab-5/blob/main/Screenshot%202026-03-14%20200306.png

2) https://github.com/Ejakosah/Lab-5/blob/main/Screenshot%202026-03-14%20203755.png

Task 3: Building the Conversion Engine (Private gRPC Service)

These screenshots captures my initial failed deployment of the conversion-engine service to Cloud Run. The container built successfully but failed to start, with logs showing an ImportError: "cannot import name 'runtime_version' from 'google.protobuf'" – this indicated a version mismatch between the protobuf library installed in the container and the version used to generate the gRPC stubs. After examining the generated conversion_pb2.py file, I discovered it required protobuf 6.31.1, but my requirements.txt specified an older version. I consulted ChatGPT and DeepSeek to understand the error pattern, and they helped me recognize this as a classic dependency compatibility issue in containerized microservices.

The solution came from updating both services' requirements.txt files to specify "protobuf==6.31.1" alongside "grpcio==1.62.1", then regenerating the stubs and rebuilding the container as version v2. This resolved the import error because the runtime environment now matched exactly what the generated code expected. The successful deployment was confirmed when curling the conversion-engine URL returned a 403 Forbidden response – exactly what we want, proving the service is properly configured as private and requires authentication. This experience demonstrated how critical precise dependency management is in microservice architectures, where even minor version mismatches can prevent services from deploying or communicating correctly.

Task 4


Task 5


Task 6


Task 7
