## GSP315 - Set Up an App Dev Environment on Google Cloud: Challenge Lab

You are just starting your junior cloud engineer role with Jooli Inc. So far, you have been helping teams create and manage Google Cloud resources. You are expected to have the skills and knowledge for these tasks, so don’t expect step-by-step guides.

## Your Challenge

You are asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called Memories. You have been asked to assist the Memories team with initial configuration for their application development environment.

You receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Function you create.
- Create a Cloud Function.
- Remove the previous cloud engineer’s access from the memories project.

Some Jooli Inc. standards you should follow:

- Create all resources in the us-west1 region and us-west1-b zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g., an instance could be named kraken-webserver1.
- Allocate cost-effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware.

These are my lab details, however, you will have your own unique ones that you can replace with.

### General Info
- **Project ID**: `qwiklabs-gcp-01-a8ef95d831ed`
- **Region**: `us-west1`
- **Zone**: `us-west1-b`

### Users
- **Username 1**: `student-04-bcf3cb8c5d80@qwiklabs.net`
- **Username 2**: `student-04-ed532d8d593e@qwiklabs.net`

### Resources
- **Bucket Name**: `qwiklabs-gcp-01-a8ef95d831ed-bucket`
- **Pub/Sub Topic**: `topic-memories-312`

### Cloud Function
- **Name**: `memories-thumbnail-creator`
- **Entry Point**: `memories-thumbnail-creator`


### Task 1: Create a Bucket

You need to create a bucket called `qwiklabs-gcp-01-a8ef95d831ed-bucket` for the storage of the photographs. Ensure the resource is created in the us-west1 region and us-west1-b zone.

#### Instructions

Go to cloud shell and run the following command to create a bucket.

```
gsutil mb gs://qwiklabs-gcp-01-a8ef95d831ed-bucket/
```

### Task 2: Create a Pub/Sub Topic

Create a Pub/Sub topic called `topic-memories-312` for the Cloud Function to send messages.

#### Instructions

Go to the Cloud Shell and run the following command to create a Pub/Sub topic:

```
gcloud pubsub topics create topic-memories-312
```
### Task 3: Create the Thumbnail Cloud Function

Create a Cloud Function named `memories-thumbnail-creator` to generate thumbnails from images added to the `qwiklabs-gcp-01-a8ef95d831ed-bucket`. Ensure the Cloud Function is utilizing the 2nd Generation environment and is created within the us-west1 region and us-west1-b zone.

#### Instructions

1. **Navigate to Cloud Functions:** Open the Cloud Console and use the Navigation menu to locate and select Cloud Functions.
2. **Create Function:** Click the "Create function" button.
3. **Configure Function:**
   - **Enviroment:** 2nd gen
   - **Function Name:** Enter `memories-thumbnail-creator`.
   - **Region:** us-west1 region
   - **Trigger:** Select Cloud Storage from the dropdown.
   - **Event Type:** Choose Finalizing/Creating.
   - **Bucket:** Specify `qwiklabs-gcp-01-a8ef95d831ed-bucket`.
   - **Maximum numbers of instances:** Type 5
4. Click "Next", then "Enable".
5. **Runtime Settings:** Choose Node.js 14 as the Runtime.
6. **Function Details:**
   - **Entry Point (Function to execute):** Type in `thumbnail`.
   - **Source Code:** Select Inline editor and replace the existing code in `index.js` and `package.json` with the code provided below.
7. In `index.js`, find and replace `REPLACE_WITH_YOUR_TOPIC_NAME` with `topic-memories-312`.

#### Add the following code to `index.js`:

```
const functions = require('@google-cloud/functions-framework');
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");

functions.cloudEvent('memories-thumbnail-creator', cloudEvent => {
  const event = cloudEvent.data;

  console.log(`Event: ${event}`);
  console.log(`Hello ${event.bucket}`);

  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "topic-memories-312";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
});
```
#### Add the following code to `package.json`:

```
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0",
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```
8. Click Deploy.

### Task 4: Test the Infrastructure

After setting up the Cloud Function and Pub/Sub topic, the next step is to verify that the infrastructure works as intended by uploading an image to the bucket and checking if a thumbnail is generated.

#### Instructions

1. **Open Cloud Storage:** In the Cloud Console, use the Navigation menu to find and open Cloud Storage.
2. **Access Your Bucket:** Click on the name of the bucket you created (`qwiklabs-gcp-01-a8ef95d831ed-bucket`).
3. **Upload an Image:**
   - In the "Objects" tab, click on "Upload files".
   - Browse your local machine in the file dialog, select a JPG or PNG image file, and confirm the upload.
4. **Refresh Bucket View:** After the upload completes, click "Refresh Bucket" to update the list of objects in the bucket.
5. **Verify Thumbnail Creation:** Look for a new object with a name similar to the uploaded image file but with a `_thumbnail` suffix. This indicates that the Cloud Function executed successfully and created a thumbnail image.

#### Troubleshooting

- If the thumbnail image does not appear, double-check the Cloud Function logs for any errors and ensure that the function is correctly triggered by uploads to the bucket.
- Ensure that the Cloud Function has the necessary permissions to access the Cloud Storage bucket and publish messages to the Pub/Sub topic.

### Task 5: Remove the Previous Cloud Engineer

As part of your onboarding tasks, you need to ensure that only current team members have access to the project's resources. This involves removing the access of the previous cloud engineer from the project.

#### Instructions

1. **Navigate to IAM & Admin:** Open the Cloud Console, use the Navigation menu to find and select "IAM & Admin".
2. **Search for the User:** In the IAM page, look for the previous cloud engineer's account (`student-04-ed532d8d593e@qwiklabs.net`) listed with the role of Viewer.
3. **Edit User Roles:** Click on the pencil icon next to the user's name to edit their roles.
4. **Remove Access:**
   - Click on the trash icon next to the Viewer role to remove it.
   - Confirm the action if prompted.
5. **Save Changes:** Ensure to click "Save" to apply the changes.
