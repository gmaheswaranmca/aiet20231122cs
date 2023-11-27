Data Source
    -> DBMS/RDBMS
        Oracle,MySQL,SQLServer,Posgres,SQLite
    -> No SQL
        Mongo DB {document}    
DB Browser for SQLite 

CREATE TABLE person(
    person_id int,
    first_name varchar(255),
    last_name varchar(255)
);

INSERT INTO person
(person_id, first_name, last_name)
VALUES(10,'Maheswaran','Govindaraju');
INSERT INTO person
(person_id, first_name, last_name)
VALUES(11,'Virat','Kohli');
INSERT INTO person
(person_id, first_name, last_name)
VALUES(12,'Rohit','Sharma');

1. $python --version
we should ve python in the comp.
2. $pip --version 
PIP : python package manager
3. $pip list
gives list of python packages installed  
if django is not there, install it.
4. $pip install django 
django is python package, 
framework for web app dev.
5. $pip install djangorestframework
6. $pip install django-cors-headers
7. $django-admin --version 
8. $django-admin --help 
"startproject","startapp","runserver"
9.E:\2019>django-admin startproject personproject
10.E:\2019>cd personproject 
11.E:\2019\personproject>code .

GET http://127.0.0.1:8000/person
GET http://127.0.0.1:8000/person/101
POST http://127.0.0.1:8000/person
{
    "person_id":101,
    "first_name":"mahesh",
    "last_name":"Govind"
}
PUT http://127.0.0.1:8000/person/101
{
    "first_name":"mahesh",
    "last_name":"Govind"
}
DELETE http://127.0.0.1:8000/person/101

$python manage.py startapp personapp

Add apps 'personapp','rest_framework' 
into INSTALLED_APPS list 
in "settings.py" 

INSTALLED_APPS = [
    ...  ,
    'personapp',
    'rest_framework',
    'corsheaders',
]

MIDDLEWARE = [
    ...,
    'django.middleware.common.CommonMiddleware',
    ...,
    'corsheaders.middleware.CorsMiddleware',
]
CORS_ORIGIN_ALLOW_URL = True

******In models.py of personapp,******
from django.db import models
from rest_framework import serializers

# Create your models here.
class Person(models.Model):
    first_name = models.CharField(
        max_length=255,
        blank=False)
    last_name = models.CharField(
        max_length=255,
        blank=False)    

class PersonSerializer(
        serializers.ModelSerializer):
    class Meta:
        model = Person
        fields = ('id',
        'first_name',
        'last_name')


$python manage.py makemigrations personapp
$python manage.py migrate 


******In personapp, "views.py":******
from django.shortcuts import render
from django.http.response import JsonResponse
from personapp.models import Person
from personapp.models import PersonSerializer
from rest_framework.decorators import api_view
from rest_framework.parsers import JSONParser
from rest_framework import status

# Create your views here.
@api_view(['GET','POST'])
def findAll(request):
    if request.method == 'GET':
        persons = Person.objects.all()
        persons_ser = PersonSerializer(persons,
            many=True)
        return JsonResponse(persons_ser.data,
            safe=False)
    elif request.method == 'POST':
        person = JSONParser().parse(request)
        person_ser = PersonSerializer(data=person)
        if person_ser.is_valid():
            person_ser.save()
            return JsonResponse(person_ser.data)
        return JsonResponse(person_ser.errors,
            status = status.HTTP_400_BAD_REQUEST)
@api_view(['GET','PUT','DELETE'])
def findById(request, id):
    if request.method == 'GET':
        person = Person.objects.get(pk=id)
        person_ser = PersonSerializer(person)
        return JsonResponse(person_ser.data,
            safe=False)
    elif request.method == 'PUT':
        old_person = Person.objects.get(pk=id)
        person = JSONParser().parse(request)
        person_ser = PersonSerializer(old_person,
                data=person)
        if person_ser.is_valid():
            person_ser.save()
            return JsonResponse(person_ser.data)
        return JsonResponse(person_ser.errors,
            status = status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE':
        old_person = Person.objects.get(pk=id)
        old_person.delete()
        return JsonResponse(
            {'message' : 
            'Person deleted successfully'})

******urls.py of personapp******
"""
URL configuration for personproject project.

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/4.2/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.urls import include, path
    2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
"""
from django.contrib import admin
from django.urls import path,re_path
from personapp.views import findAll
from personapp.views import findById

urlpatterns = [
    path('admin/', admin.site.urls),
    path('person/',findAll),
    re_path(r'^person/(?P<id>[0-9]+)$', findById)
]


