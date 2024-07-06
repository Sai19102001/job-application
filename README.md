# job-application
Backend (Django with MySQL)

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'your_database_name',
        'USER': 'your_database_user',
        'PASSWORD': 'your_database_password',
        'HOST': 'your_database_host',
        'PORT': 'your_database_port',
    }
}
3. Create Models
In jobs/models.py, define models for User, JobListing, and Application.
python
Copy code
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    pass

class JobListing(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
    company = models.CharField(max_length=255)
    location = models.CharField(max_length=255)
    posted_at = models.DateTimeField(auto_now_add=True)

class Application(models.Model):
    job = models.ForeignKey(JobListing, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    cover_letter = models.TextField()
    status = models.CharField(max_length=50, choices=[('applied', 'Applied'), ('reviewed', 'Reviewed'), ('rejected', 'Rejected'), ('accepted', 'Accepted')])
    applied_at = models.DateTimeField(auto_now_add=True)
4. User Authentication
Use Django’s built-in authentication system for user registration, login, and management.
5. Develop API Endpoints
Use Django REST framework to create API endpoints.
bash
Copy code
pip install djangorestframework
In jobs/views.py, create views for handling API requests.
python
Copy code
from rest_framework import viewsets
from .models import JobListing, Application
from .serializers import JobListingSerializer, ApplicationSerializer

class JobListingViewSet(viewsets.ModelViewSet):
    queryset = JobListing.objects.all()
    serializer_class = JobListingSerializer

class ApplicationViewSet(viewsets.ModelViewSet):
    queryset = Application.objects.all()
    serializer_class = ApplicationSerializer
In jobs/serializers.py, define serializers for the models.
python
Copy code
from rest_framework import serializers
from .models import JobListing, Application

class JobListingSerializer(serializers.ModelSerializer):
    class Meta:
        model = JobListing
        fields = '_all_'

class ApplicationSerializer(serializers.ModelSerializer):
    class Meta:
        model = Application
        fields = '_all_'
In jobboard/urls.py, register the viewsets with a router.
python
Copy code
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from jobs.views import JobListingViewSet, ApplicationViewSet

router = DefaultRouter()
router.register(r'jobs', JobListingViewSet)
router.register(r'applications', ApplicationViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
Frontend (Next.js)
1. Set Up a Next.js Project
Initialize a new Next.js project.
bash
Copy code
npx create-next-app job-board-frontend
cd job-board-frontend
2. Create Pages
Create pages for user registration, login, job listings, job details, job application, and application status.
3. Implement State Management
Use a state management library like Redux or React Context API.
bash
Copy code
npm install @reduxjs/toolkit react-redux
Set up a Redux store for managing user authentication and job application operations.
Example Directory Structure:
bash
Copy code
pages/
  index.js
  login.js
  register.js
  jobs/
    index.js
    [id].js
  applications/
    index.js
store/
  index.js
  userSlice.js
  jobSlice.js
components/
  JobList.js
  JobDetails.js
  ApplicationForm.js
Example Code Snippet
store/userSlice.js
javascript
Copy code
import { createSlice } from '@reduxjs/toolkit';

export const userSlice = createSlice({
  name: 'user',
  initialState: {
    isAuthenticated: false,
    user: null,
  },
  reducers: {
    login: (state, action) => {
      state.isAuthenticated = true;
      state.user = action.payload;
    },
    logout: state => {
      state.isAuthenticated = false;
      state.user = null;
    },
  },
});

export const { login, logout } = userSlice.actions;
export default userSlice.reducer;
store/jobSlice.js
javascript
Copy code
import { createSlice } from '@reduxjs/toolkit';

export const jobSlice = createSlice({
  name: 'jobs',
  initialState: {
    listings: [],
    applications: [],
  },
  reducers: {
    setJobListings: (state, action) => {
      state.listings = action.payload;
    },
    setApplications: (state, action) => {
      state.applications = action.payload;
    },
  },
});

export const { setJobListings, setApplications } = jobSlice.actions;
export default jobSlice.reducer;
Example API Call
pages/jobs/index.js
javascript
Copy code
import { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { setJobListings } from '../../store/jobSlice';

const JobList = () => {
  const dispatch = useDispatch();
  const jobListings = useSelector(state => state.jobs.listings);

  useEffect(() => {
    fetch('/api/jobs')
      .then(response => response.json())
      .then(data => {
        dispatch(setJobListings(data));
      });
  }, [dispatch]);

  return (
    <div>
      <h1>Job Listings</h1>
      <ul>
        {jobListings.map(job => (
          <li key={job.id}>{job.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default JobList;
