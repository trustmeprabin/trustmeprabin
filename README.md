- 👋 Hi, I’m @trustmeprabin
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
trustmeprabin/trustmeprabin is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
from django.db import models
from django.contrib.auth.models import User

# Create your models here.
class Company(models.Model):
    user=models.OneToOneField(User,null=True,on_delete=models.CASCADE)
    name=models.CharField(max_length=200,null=True)
    position=models.CharField(max_length=200,null=True)
    description=models.CharField(max_length=2000,null=True)
    salary=models.IntegerField(null=True)
    experience=models.IntegerField(null=True)
    Location=models.CharField(max_length=2000,null=True)

    def __str__(self):
        return self.name


class Candidates(models.Model):
    category=(
        ('Male','male'),
        ('Female','female'),
        ('Other','other'),
    )

    name=models.CharField(max_length=200,null=True)
    dob=models.DateField(null=True)
    gender= models.CharField(max_length=200,null=True,choices=category)
    mobile= models.CharField(max_length=200,null=True)
    email= models.CharField(max_length=200,null=True)
    resume=models.FileField(null=True)
    company=models.ManyToManyField(Company,blank=True)

    def __str__(self):
        return self.namefrom django.forms import ModelForm
from .models import *

class ApplyForm(ModelForm):
    class Meta:
        model=Candidates
        fields="__all__"from django.forms import ModelForm
from .models import *

class ApplyForm(ModelForm):
    class Meta:
        model=Candidates
        fields="__all__"from django.contrib import admin
from .models import *

# Register your models here.
admin.site.register(Company)
admin.site.register(Candidates)STATIC_URL = '/static/'
MEDIA_URL='/media/'
MEDIA_ROOT=os.path.join(BASE_DIR,'media/')from django.urls import path
from .views import *

urlpatterns = [
    path('',home,name='home'),
    path('login/',loginUser,name='login'),
    path('logout/',logoutUser,name='logout'),
    path('register/',registerUser,name='register'),
    path('apply/',applyPage,name='apply'),
]from django.shortcuts import render,redirect
from .models import *
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth import login,logout,authenticate
from .forms import *

# Create your views here.
def home(request):
    if request.user.is_authenticated:
        candidates=Candidates.objects.filter(company__name=request.user.username)
        context={
            'candidates':candidates,
        }
        return render(request,'hr.html',context)
    else:
        companies=Company.objects.all()
        context={
            'companies':companies,
        }
        return render(request,'Jobseeker.html',context)


def logoutUser(request):
    logout(request)
    return redirect('login')


def loginUser(request):
    if request.user.is_authenticated:
        return redirect('home')
    else:
       if request.method=="POST":
        name=request.POST.get('username')
        pwd=request.POST.get('password')
        user=authenticate(request,username=name,password=pwd)

        if user is not None:
            login(request,user)
            return redirect('home')
       return render(request,'login.html')


def registerUser(request):
    if request.user.is_authenticated:
        return redirect('home')
    else:
        Form=UserCreationForm()
        if request.method=='POST':
            Form=UserCreationForm(request.POST)

            if Form.is_valid():
                currUser=Form.save()
                Company.objects.create(user=currUser,name=currUser.username)
                return redirect('login')
        context={
            'form':Form
        }
        return render(request,'register.html',context)


def applyPage(request):
    form=ApplyForm()
    if request.method=='POST':
        form=ApplyForm(request.POST,request.FILES)

        if form.is_valid():
            form.save()
            return redirect('home')
    context={'form':form}
    return render(request,'apply.html',context){% extends 'main.html' %}
{% load static%}

{% block content %}
<div class="container card">
    <h1>Candidates applied</h1>
    <div class="card-body">
        <table class="table table-hover">
            <tr>
                <th>Name</th>
                <th>DOB</th>
                <th>Resume</th>
                <th>Gender</th>
                <th>Mobile</th>
                <th>Email</th>
            </tr>
            {% for c in candidates %}
                <tr>
                    <td>{{c.name}}</td>
                    <td>{{c.dob}}</td>
                    {% if c.resume %}
                    <td><a href='{{c.resume.url}}' download >{{c.resume}}</td></a></td>
                    {% else %}
                    <td>Not submitted</td>
                    {% endif %}
                    <td>{{c.gender}}</td>
                    <td>{{c.mobile}}</td>
                    <td>{{c.email}}</td>
                </tr>
            {% endfor %}
        </table>
    </div>
</div>

{% endblock %}{% extends 'main.html' %}
{% load static%}

{% block content %}
<div class="container card">
    <h1>Available openings</h1>
    <div class="card-body">
        <table class="table table-hover">
            <tr>
                <th>Name</th>
                <th>Position</th>
                <th>Description</th>
                <th>Salary</th>
                <th>Experience</th>
                <th>Location</th>
                <th>Join</th>
            </tr>
            {% for c in companies %}
                <tr>
                    <td>{{c.name}}</td>
                    <td>{{c.position}}</td>
                    <td>{{c.description}}</td>
                    <td>{{c.salary}}</td>
                    <td>{{c.experience}}</td>
                    <td>{{c.Location}}</td>
                    <td><a href="{% url 'apply' %}" class="btn btn-info btn-sm" type="submit">Apply</a></td>
                </tr>
            {% endfor %}
        </table>
    </div>
</div>
        {% for c in companies %}
            {{c.name}}
        {% endfor %}
{% endblock %}{% extends 'main.html' %}
{% load static%}

{% block content %}
<head>
    <style>
        .box{
            border:5px solid black;
        }
    </style>
</head>
<div class=" box jumbotron container">
    <form action="" method="POST" enctype="multipart/form-data">
        {% csrf_token %}
        {{form.as_p}}
        <input type="submit" class="btn btn-success btn-sm" value="submit">
    </form>
</div>
{% endblock %}
