# My development stack

I always try to improve my development stack in order to gain productivity. And during the last few months, there has been a lot of change. 

Here is my current development stack that I use to create web apps.

###### Front-end:
**Bootstrap:** A great HTML/CSS framework. It helps you build MVC very fast.  
**Sass:** A language compiled into CSS which adds many features such as variables, imports and functions.  
**Jquery:** Good old Jquery, indeed. I may start to use angular in the next few weeks if I have enough time.

###### Back-end:
**Ruby on Rails:** The most productive web framework I have ever worked with, much faster than any PHP framework, and enforcing many good software engineering practices.  
**PostgreSQL:** Faster and most powerful than most RDBMS (including MySQL), and free.  

###### Hosting:
**Heroku:** A great PaaS, and free for small apps. I also tried Google App Engine but its lack of flexibility was a problem for me.  
**Cloudflare:** It makes your app load faster and gives you a *free SSL certificate*.  
**Mandrill:** Very easy to use, yet powerful, EaaS (Email as a Service). I use it to send transactional emails like welcome messages and password resets.

###### Deployment:
**Git:** I configured it with a pre-commit hook which automatically runs rails tests. And thanks to Heroku a simple push deploys my code. It’s very useful and now I just couldn’t go back. 

###### Monitoring:

**Pingdom:** A great tool to receive alerts if your app goes down (and if you add [IFTTT](https://ifttt.com/) you can receive them by SMS).

Let’s see what it looks like (with the help of a great website I recently found):

![alt](http://i.imgur.com/tQUt7cR.png)
http://stackshare.io/berpj/my-development-stack

That’s it for now!