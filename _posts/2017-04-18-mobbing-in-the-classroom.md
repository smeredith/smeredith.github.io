---
layout: post
title: Mob Programming in the Classroom
---

I've been teaching Python to a high school class over a video link as a volunteer for about 2 months.
We were stuck in a rut, so I decided to shake it up a little and introduce mob programming.
I didn't tell them it was mob programming, but instead described how we were doing to complete an assignment as a group.

There are 9 students in the class, arranged in two rows.
The classroom teacher sits behind and to the side.
She is learning Python along with her students.
There is a video camera in the front so that I can see the class.
We use Skype for business for me to deliver the lessons and for the students to get one-on-one help with labs.
They log on to Skype individually from their own desk.
We use Cloud9 as our Python IDE.

## Day 1

Each student was at his own desk and logged into Skype, with headphones on.
Normally, all mics are muted unless somebody has something to say.
I instructed them all to unmute.
I told the students to access my shared Cloud9 workspace.
With Cloud9, everybody can edit the source file simultaneously, and we all opened the same file.

I described that we would have a driver at the keyboard and a navigator to tell the driver what to type, and that only the driver may type.
I told them that the rest of the class would be the brain and that the navigator would ask the brain for ideas on what to tell the driver to type.
I selected a driver and a navigator sitting next to each other.
I set a timer to rotate them every 4 minutes.
The teacher was to participate as one of the students.
A TA (also remote) and I were available for questions, but we were not needed.

Once we started, the first thing we did was abandon the headsets.
Skype is not robust enough for multiple people to be talking at the same time.
After that, I could hear the students through the video camera at the front of the class, but they could not hear me.
I used Cloud9 chat and waved my arms around in Skype to tell them to rotate.

We had enough time for everybody to be the navigator and the driver exactly once.
Since each workstation has the file open for editing, each driver entered text from his own desk.
I noticed that since the driver and the navigator were sitting close together, most of the discussion was between those two, leaving out most of the rest of the class.
When the rotation got around to the teacher, she had the students gather around her screen and the discussion was more inclusive.
In the future, I will start the navigator and the driver at opposite ends of the classroom.

A four minute rotation felt a little too long, as attention of the brain seemed to wander at times.
I will shorten this to three minutes next time to see if that makes it feel more lively.

We had a short retro at the end of the class.
The students seemed to enjoy the exercise, but felt that the classroom is not well configured for it.
Some complained of lag with the Cloud9 IDE.
One student actually could not enter text, so he missed his turn as driver and the next person in the rotation did double duty.
One said that he liked it because they could make progress without getting stuck.
Another said groups of 3 might work better.
The teacher thought 10 minutes would be a better rotation time, particularly if they worked in groups of three.

The students will all get the same grade on this assignment.

## Day 2

We started a new problem.
We kept everything the same as yesterday except I shortened the rotation time to 3 minutes and I assigned the driver and navigator to people sitting in different parts of the room.
The students knew the drill and fell into a rhythm quite quickly.
Most gathered around the workstation of the driver, but a couple watched from their own desk.

At the point the students started making noises about being done with the lab, I pointed out a few areas where they didn't quite meet the requirements.
They started changing the code but didn't finish the assignment before running out of time.

I didn't allow enough time for a retrospective today but I wish I had.
I felt that the three minute timer worked better than the four.
In the future, I would like to try a configuration with a projector and everybody rotating though the teacher's workstation, since hers is connected to the  projector.

In this volunteer teaching arrangement, we rotate between two volunteer teachers on a two-day-on, two-day-off schedule.
I am off tomorrow, and I have shared the experience of these two days with my co-teacher in an after-action report.
I suggested that he allow the students to finish this lab as a mob.
I am leaving it up to him whether he wants to continue like this for the next lab.
