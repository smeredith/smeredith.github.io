---
layout: post
title: Flickr and AWS
---

When Flickr announced that they were giving everyone a terabyte of space for their photos, I decided to copy mine to the service.
Since I have almost 60GB of 32,000 photos, uploading them from my PC would have taken a long time.
Since I already had a backup of my photos on AWS S3, I decided to write a distributed app to run on AWS EC2 to copy the photos from S3 to Flickr.
Beyond S3 and EC2, this app also makes use of Amazon's queuing services: SQS.
In addition to parallelizing the work, the app takes advantage of the high bandwidth available to EC2 instances.
By using this technique, the time it took to upload the files was reduced from days to hours, with the potential for further reduction by running more worker instances.

## Step 1: Populate SQS

I wrote a Windows console app using C# and Amazon's own AWS .NET library.
This app queries S3 for the file metadata of all the photos, and fills an SQS queue with one message for each file.
The message contains the URL to the file on S3, and a retry counter.
I like JSON for the body of message queue messages because it is compact, human-readable, and easy to parse.

## Step 2: Drain SQS and Transfer to Flickr

I wrote a second console app using C#, Amazon's library, and FlickrNet .NET library for using the Flickr API.
The idea here is to run multiple instances of this app simultaneously on multiple EC2 virtual machines.
Each instance grabs a message from the queue and transfers that file to Flickr.
If the transfer fails, the message is put back on the queue with a decremented retry count.
If the retry count reaches 0, that file is reported as failed.
As a feature of SQS, if the app crashes or the VM dies while processing a message, that message is automatically returned to the queue after some period of time.

Unfortunately, there is no way to directly transfer the file from S3 to Flickr.
The file has to make an temporary stop on the machine running the app.
So the transfer of a single file consists of downloading the file from S3 to the local file system, and then uploading the file to Flickr.
Remember that when the app is running on EC2, the first transfer is AWS to AWS.

Flickr has the notion of a "set," which is essentially an album.
There is no hierarchy to sets.
Once the file has been uploaded to Flickr, the app puts it into a set that has a name that reflects the directory name of the original file.
A set is created after the first time a photo that should belong to that set is uploaded.
I discovered a race condition bug in my app here.
Multiple instances of the app created multiple sets with the same name, which is allowed on Flickr.
This bug was made more frequent by the fact that the files were sorted by full path and filename when they entered the queue.
I had to write another simple app later to merge duplicate Flickr sets.
If I were to do it again, I would manage putting photos into sets serially after all the files where finished uploaded.
All the information needed to create the set is already encoded in the name of the photo on Flickr, so at this point, I wouldn't need any of the original metadata from S3.
If I had needed the metadata, I would have created another queue, and created a single worker to service that queue sequentially to avoid the race condition.

## Step 3: Deploy to EC2 and Let It Rip

During the development, I could run everything on my local machine.
Access to SQS and S3 is no problem (it's just HTTP,) and I could test and debug as needed.
In theory, I could get the job done by letting a single instance run for a long time on my local machine.
When the app was ready, I prepared an EC2 instance using a standard Windows Server AMI.
I customized this to run the Step 2 app on startup.
Then I saved this image, and launched ten EC2 (micro) instances of it.
The apps did their work, and in a few hours, the queue was empty and all the files were on Flickr.
More instances would have completed the job ever faster.

## Why not Azure?

Why did I use AWS instead of Azure for this?
Because I already had the files on S3.
It would have been just as easy to use Azure queues and Azure VMs.
If the files were on Azure blob storage instead, I would have.
In fact, you can mix and match these services.
I could have run the Step 2 app on Azure VMs unmodified.
Or I could have run some instances on EC2 and some on Azure simultaneously.
Or I could have used Azure queues instead of SQS, and run the app on either EC2 or Azure VMs.

## What's Next?

I have since written more code to keep the Flickr photos in sync with my local photos, which I consider the master set.
AWS does not come into the picture with that app: I'm only uploading new and changed files, so there would be no benefit.
If I ever have a catastrophic systems failure and I lose the photos from both the local file system and S3, I can restore from Flickr.
The files on Flickr are bit-for-bit copies of the originals, and I can rebuild the local directory structure from the names of the files or the names of the sets.

## Conclusion

The app was a success: it did its work reliably and quickly, and now all my photos are on Flickr.
It was easy, and C# is perfect for this sort of thing.
There was no real urgency for me to move the photos to the service, so this was mostly a fun exercise.
However, if there had been a deadline, or if the size of the data set was significantly larger, this might have become more than an exercise.
The cool part is the scalability: the time to complete the work is proportional to the number of instances you spin up.
I now have a proven, easy approach for the next time I need to solve a problem like this.

I really like AWS.
The services are well thought-out and the APIs are easy to understand and use.
It is affordable to play with.
This isn't my first distributed app using the services, but it is the most rewarding because of the tangible results.

I also like Flickr.
It has been getting some bad press since its recent redesign, but I enjoy using it to view my photos, and you can't complain about the amount of free storage.
The Windows Phone Flickr app needs some work, but the iOS version is pretty good.
It's great to have access to every single photo I own from a mobile device without taking up local storage.
I can't view them while the phone is disconnected, but that is not a hardship.

This technique will work equally well to quickly transfer files between any two cloud storage providers.
You just need to swap out calls to the Flickr API with calls to your target cloud provider.
If you abstract away the details of the provider, the swap should be pretty easy.
