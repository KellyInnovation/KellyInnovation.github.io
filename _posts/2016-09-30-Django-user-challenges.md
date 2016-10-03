---
layout: post
title: Branching Out and Django Users
---

We are continuing with Django this week.  This class is pushing me to new levels of online presence I had never expected.  We started the class by signing up for Trello, Slack, StackOverflow, and GitHub.  Before starting class, it was an introvert’s challenge for me to just slack a link to my GitHub page, much less post a non-prework comment.  

Bootcamp may have started with slightly uncomfortable slack posts, but has since escalated to more social sharing.  As part of the bootcamp, we are required to post a blog weekly.  Knowing this made me briefly reconsider this adventure.  While still not my favorite part, I am acclimating to writing blogs.  The current blog challenge is no longer that it will posted to the internet forever, but merely determining a topic.  

This week, I took one of the last big steps in my social presence: I joined Twitter.  Our first big portfolio piece is to replicate Twitter.  Not having used it before, I needed to join to see all aspects of the site to better complete my assignment.  (Twitter is not two birds having an argument about books at the library, as my daughter asserted.)

After initial project planning, I started with an index.html page and a registration form.  I wanted to be able to log in users.  That went well and relatively quickly, but then arose the problem of accessing the user data.  Django is nice enough to provide a User Model in which validation and password security are built in.  

The biggest problem has been getting the User information and using it in other apps.  In views:
	from django.contrib.auth.models import User

	def profile_list(request):
		users = User.objects.all()

	context = {“users”:users,}

	return render(request, “user_profiles/profile.html”, context)
In profile.html:
	{% for user in users %}
		{{user.username}}
	{% endfor %}

So few lines of code, but so many hours to get there (with help).

Interestingly, my biggest struggles with this project are all related to the User model.  The next problem was connecting an individual tweet to the logged in user.  The fix is to add the user before saving the tweet:
	tweet_text = form.save(commit=False)
	tweet_text.user = request.user
	tweet_text.save()

While the User app presented challenges, I am glad it is built in because of its power.