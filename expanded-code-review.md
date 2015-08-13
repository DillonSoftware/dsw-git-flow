### expanded code reviews

This was discussed @ dilloncon, but we were near the end of the week, and some people were already gone, so I just wanted to reiterate what we discussed.

While reviews are valuable, they are a bottleneck now because we only have a couple of people doing them.

Rather than abandoning the idea to speed things up, we're going to spread the work out to speed it up.

When you complete a task that you are working on, the basic process is:

 - pull the latest 'develop' branch down
 - merge the 'develop' branch into your branch and make sure that it's still good
 - push the changes up
 - mark the task as 'written'

At this point, it's assumed that you have:

 - checked the changes for conventions/style (IDEA and checkstyle will point out most oddities in you code - look at the checkstyle report and the right margin in IDEA for red/yellow marks)
 - verified that the code does what the task asked for (if it doesn't because the task is wrong, add a comment to indicate that)
 - added steps to test the task to the comments on the task so that the reviewer and QC people can verify the change does what it's is supposed to

BEFORE looking for your next 'new' task, look for other 'written' tasks that are not yours and that you can review. If there is a task to be reviewed, mark it as 'in review' so no one else starts reviewing it, and then do the review on it following the process outlined in our "review-merge-process.md" document.


