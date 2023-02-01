# Django for Rails Developers

For the past 3 years, I’ve worked on both Rails and Django. Though Rails is my favorite (Yes, Swaathi made me write this), I would like to show you how similar and sometimes strikingly different both are. This article helps any Rails dev easily port over to Django. Well, what are we waiting for, let’s get started! :D

### Installation:

Make sure you have Python (at least 3) installed.
Then just do this,

<table>
<tr>
<td>Rails
</td>
<td>Django
</td>
</tr>
<tr>
<td><code>gem install rails</code>
</td>
<td><code>pip install django</code>
</td>
</tr>
</table>

So similar!

### Architecture:

Rails follows the MVC (Model, View, Controller) architecture. The Model is a representation of the database structure that is encapsulated in the host language. The Controller acts as an interface between the outside and inside world of a web framework. This is basically where requests are received and processed before responding. The View is a template that is shown to the user.

Django follows MVT (Model, View, Template) architecture. Here the view acts as the Rails controller and the template acts as the Rails view - I know, confusing!

### Create App:

Rails uses “app” to represent the working directory. Whereas in Django, “project” is used.

<table>
<tr>
<td>Rails
</td>
<td>Django
</td>
</tr>
<tr>
<td><code>rails new <app-name></code>
</td>
<td><code>django-admin startproject <app-name></code>
</td>
</tr>
</table>

### Run Development Server:

Both Rails and Django come bundled with a local web server.  The default port is 8000 in Django and 3000 in Rails. The Django server is a light weight server purely written in Python where as in Rails the server is a Puma web server.

<table>
<tr>
<td>Rails
</td>
<td>Django
</td>
</tr>
<tr>
<td><code>rails server</code>
</td>
<td><code>python manage.py runserver</code>
</td>
</tr>
</table>

### Settings.py:

Settings.py is similar to application.rb where we have the configurations of the project. In Django, all the configurations like database connection, timezone, template location, allowed hosts and so on are added to settings.py.

### Manage.py:

Manage.py is the most important file when it comes to Django. This manage.py has the core functions required like makemigrations, migrate, runserver and so on.

### Migrations:

Unlike Rails the migrations in Django need two steps. Those are “make migration” and “run migration”. In Rails, we create migrations for the models at the entire application level. But in Django we need to write all the fields required and the models methods in the model.py of the corresponding app.

Note: Initially, we need to run the migrate command to run the default migrations.

<table>
<tr>
<td>Rails
</td>
<td>Django
</td>
</tr>
<tr>
<td><code>rails db:migrate</code>
</td>
<td><code>python manage.py makemigrations</code>
<p>
<code>python manage.py migrate</code>
</td>
</tr>
</table>

### Models:

Both in Rails and Django models function as the same except for the fact that the fields of the model have to be mentioned in the model file in Django. But similar to Rails, we include the model methods and validations in the model file. In Django, model files are created in the app’s root directory.

### View:

In Django views are similar to the controller in Rails. They act as the interface between the model and template. We define the actions in the views.py in the corresponding app directory similarly like controllers in rails.

### Template:

In Django templates are similar as Views in Rails. In Django, Jinja2 is followed for the template format. Here all the templates are kept, in the template folder of each app’s root directory. Jinja2 is a template engine user in Django. Each view in Django returns an object which contains the data required in the template. Only the data available in the return object is accessible by the template, like the instance variable in rails.

The basic elements in Jinja 2 are
```
1. {%....%} are for statements
2. {{....}} are expressions used to print to template output
3. {#....#} are for comments which are not included in the template output
4. #....## are used as line statements
```
### URLs:

Urls.py are more like routes.rb in Rails. In Django, the URL specifies the view function for each path and action. The URLs corresponding to each app is available in the app’s root directory. Don’t forget to import views in the urls.py in the main project urls.py.

One of the main changes, that has to be considered while working on Django is that we need to import the necessary library files in each model, view, and URL, unlike in Rails where everything is included when the app executes.

I hope this gives a good idea for a Rails developer to get started with Django! Try it out and let me know what you think! :)