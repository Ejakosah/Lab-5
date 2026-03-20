# Lab-5
Task 1: Environment Setup

![Screenshot](Screenshot1a.png)
This screenshot shows the successful cloning of the lab5-unit-conversion repository and that I made the python environment. The ls command confirms all required directories were present. The gcloud services enable command successfully enabled all required APIs. All foundational services necessary for deploying serverless containers were made.

Task 2: Generate gRPC Stubs from the Contract
These screenshots show the execution of `bash setup.sh` which runs the protoc compiler to generate Python gRPC stubs. The generated files are shown in both the conversion-engine and converter-api directories. This process enforces the service contract making sure that both services work together on data structure when compiling instead of when it hits runtime. 

Task 3: Building the Conversion Engine (Private gRPC Service)

These screenshots captures my initial failed deployment of the conversion-engine service to Cloud Run. The container built successfully but failed to start, with logs showing an ImportError: "cannot import name 'runtime_version' from 'google.protobuf'" – this indicated a version mismatch between the protobuf library installed in the container and the version used to generate the gRPC stubs. After examining the generated conversion_pb2.py file, I discovered it required protobuf 6.31.1, but my requirements.txt specified an older version. I consulted ChatGPT and DeepSeek to understand the error pattern, and they helped me recognize this as a classic dependency compatibility issue in containerized microservices.

The solution came from updating both services' requirements.txt files to specify "protobuf==6.31.1" alongside "grpcio==1.62.1", then regenerating the stubs and rebuilding the container as version v2. This resolved the import error because the runtime environment now matched exactly what the generated code expected. The successful deployment was confirmed when curling the conversion-engine URL returned a 403 Forbidden response – exactly what we want, proving the service is properly configured as private and requires authentication. This experience demonstrated how critical precise dependency management is in microservice architectures, where even minor version mismatches can prevent services from deploying or communicating correctly.

Task 4: Build the Converter API

This captures the successful build and deployment of the converter-api service. The `gcloud builds submit` command uploaded the source code to Cloud Build. During this process the command which built the container pushed it to Artifact Registry. The `gcloud run deploy` command created a new Cloud Run service making it publicly accessible from the internet. 

Task 5: Grant Service-to-Service Permissions


This screenshot shows the IAM policy binding that grants the converter-api permission to invoke the private conversion-engine. The `gcloud run services add-iam-policy-binding` command attaches the `roles/run.invoker` role to the service account for the conversion-engine. The verification command shows the binding was successfully added and the output confirmed the service account is authorized to call the private engine. This implements the zero-trust security model where the services have to prove identity rather than just relying only on network location which is more secure.

Task 6: Test the End-to-End System
There was a successful health check returning {"status": "ok"} and the units endpoint listing all five conversion categories (length, weight, temperature, speed, volume) with their units. This is showing that the public API is accessible and properly configured. KM to miles also worked. Weight to kg also worked. Each response confirms the gRPC call to the conversion engine. This shows the engine handles both linear conversions and non-linear conversions. The API rejected an invalid cross-category conversion request (miles to kg). The response returns HTTP 400 with the error message "Cannot convert miles (length) to kg (weight)" and a hint that units must be in the same category. This shows the converter-api performs validation at the system boundary, preventing invalid requests from ever reaching the conversion engine. The internal service never receives malformed input. A 403 Forbidden error from Cloud Run's edge layer occurred as a result of curling the conversion-engine URL directly. Cloud Run rejects any request that doesn't include a valid identity token from an authorized service account, demonstrating platform-level security enforcement despite being accessible via URl.

Task 7: Explore Google Cloud Console
What’s shown is both Cloud Run services (converter-api and  conversion-engine) in the Google Cloud Console dashboard with green  checkmarks showing they are healthy. The converter-api Security tab  displays "Allow unauthenticated" with an asterisk confirming allUsers  has the invoker role, making it publicly accessible as intended. The  conversion-engine Security tab shows "Require authentication" with IAM,  and the policy lists the service account 375201974496-compute@developer.gserviceaccount.com  with the Cloud Run Invoker role. The logs tab for each service shows the HTTP requests and  gRPC calls, confirming the separation between  the public API and private engine. The Artifact Registry view displays  both container images with their digests and push dates, showing that immutable container images were built before deployment.

Reflection Questions
