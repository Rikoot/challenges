## Web Discovery With Ffuf
Practice introductory Fuzzing Techniques against a dockized application. 

For this challenge, you are tasked with discovering five flags on the docker web site using `ffuf`. The flags are in the following format: `ffuf{<TEXT>}`.

Need help? See the official writeup in [`WRITEUP.md`](WRITEUP.md).

---
#### Setup Instructions
Download and unzip `Web_Discovery_with_Ffuf.zip`.  After installing docker, run the following command in the unzipped directory:
```
docker compose up 
```
You will be able to access the website on `localhost:8080`. See line `20` in the `docker-compose.yaml` file to change this default mapping. 