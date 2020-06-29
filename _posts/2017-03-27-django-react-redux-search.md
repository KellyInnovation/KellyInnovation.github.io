---
layout: post
title: Django React Redux Searches
---
The goal is to create a hobby board that connects people with similar hobbies and locations.

This will be with a Django backend and a React / Redux frontend.

The format is a table with hobbies, categories, cities, and states.
In order to be useful, tables should be searchable by hobby, category, city, or state.
How to create a table of information with a search option is the intent of this post.


##**Model**:

To start the project, begin by creating the models needed.  Create one model for additional profile information associated with a given user.  To connect those with similar hobbies within a region, include city and state in the profile.  

The OneToOneField to User (built into Django) connects profile to a first name, last name, and email address, thereby providing additional, future connectivity.

On the hobby model, include a name for the hobby, the category of the hobby, a description and a ForeignKey to the profile.  The ForeignKey connects hobby to the profile and allows for multiple hobbies to attach to one profile (allowing for one person to have many hobbies).


``` [python]
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    AGE_RANGE = (
        ('UNDER_25', 'Under 25'),
        ('26_45', '26 - 45'),
        ('46_55', '46 - 55'),
        ('56_65', '56 - 65'),
        ('OVER_65', 'Over 65'),
        ('NO_ANSWER', 'Prefer Not to Answer'),
    )

    GENDER_CHOICES = (
        ('MALE', 'Male'),
        ('FEMALE', 'Female'),
        ('SELF_DESCRIBE', 'Self Describe'),
        ('NO_ANSWER', 'Prefer Not to Answer'),
    )

    user = models.OneToOneField(User, blank=True, null=True)
    city = models.CharField('city', max_length = 50, null=True)
    state = models.CharField('state', max_length=50, null=True)
    zipcode = models.CharField('zip code', max_length=10, null=True)    
    
    age_distribution = models.CharField('age distribution', max_length=100, choices=AGE_RANGE, default='NO_ANSWER')
    gender = models.CharField('gender', max_length=100, choices=GENDER_CHOICES, default='NO_ANSWER')

class Hobby(models.Model):
    hobby_profile = models.ForeignKey(Profile, verbose_name='profile', blank=True, null=True, related_name='hobby')
    name = models.CharField('hobby name', max_length=200, null=True, blank=True)
    category = models.CharField('hobby category', max_length=200, null=True, blank=True)
    description = models.TextField('description', blank=True, null=True)
```


##**Serializer**: 

Create Serializers to connect to the frontend.
In the Serializers, list all fields in the model and define additional ones needed for GET, POST, or PUT requests.  

While hobby_profile is a field on Hobby, and can therefore be listed as a field, there is not a similar field on Profile.  In order to list hobby as a field on Profile, create HobbyField.  This assigns the associated Hobby object to the profile, so the data is accessible via Profile.

The HobbyField creates a way to access information about hobbies from the profile state.  For example, on the front end, profile.hobby.name will return the name of a hobby associated with that profile without additional API calls.


``` [python]
from django.contrib.auth.models import User
from rest_framework import serializers

from .models import *

class HobbySerializer(serializers.ModelSerializer):

    class Meta:
        model = Hobby
        fields = (
            'id',
            'hobby_profile',
            'name',
            'category',
            'description',
        )

class HobbyField(serializers.Field):
    def to_internal_value(self, data):
        try:
            return Hobby.objects.get(id=data)
        except:
            return None

    def to_representation(self, obj):
        return HobbySerializer(obj).data

class ProfileSerializer(serializers.ModelSerializer):
    hobby = HobbyField(required=False)

    class Meta:
        model = Profile
        fields = (
            'id',
            'user',
            'city', 
            'state', 
            'zipcode',

            'age_distribution',
            'gender',

            'hobby',
        )

class CurrentUserSerializer(serializers.ModelSerializer):

    class Meta:
        model = User
        fields = (
            'id',
            'username',
            'email',
            'first_name',
            'last_name',
        )
```


##**URLs for API**:

In order to connect the Django backend to the React / Redux front end, create urls for the Django, API app.

'profile' and 'hobby' are utilized on the frontend in the urls of the api calls.


``` [python]
from django.conf.urls import include, url
from rest_framework import routers

from profile.views import  CurrentUserDetails
from profile.viewsets import *

router = routers.DefaultRouter()

router.register(r'profile', ProfileViewSet)
router.register(r'hobby', HobbyViewSet)

urlpatterns = [
    url(r'^me/$', CurrentUserDetails.as_view(), name="me"),
    url(r'^', include(router.urls)),
]
```


##**apiActions**:

On the frontend, these API calls are made in actions.

Each API call from the frontend to the backend requires a unique action function to designate the type of call (the API_CALL constant functionality was defined in middleware - not shown); the endpoint (a unique constant); and the url (which defines which backend connections to make).


``` [javascript]
import {
  API_CALL,
} from '../constants/actionTypes';

import {    
    GET_PROFILES,
    GET_HOBBIES,
    GET_PROFILE_SEARCH,
} from '../constants/apiConstants';

export function getProfiles() {
    return {
        type: API_CALL,
        endpoint: GET_PROFILES,
        url: '/profile/',
    };
}

export function getHobbies() {
    return {
        type: API_CALL,
        endpoint: GET_HOBBIES,
        url: '/hobby/',
    };
}

export function getProfileSearch(query) {
    return {
        type: API_CALL,
        endpoint: GET_PROFILE_SEARCH,
        url: `/profile/?query=${query}`,
    };
}
```


##**apiConstants**:

Constants connect the various attributes associated with their use (actions and reducers).


``` [javascript]
export const BASE_URL = '/api';

export const GET_PROFILES = 'GET_PROFILES';
export const GET_HOBBIES = 'GET_HOBBIES';
export const GET_PROFILE_SEARCH = 'GET_PROFILE_SEARCH';
```


##**Reducers**:


The profile and hobbies states seen in the Component come from the reducers.

In each reducer, list all constants associated with that state. For example, in profile, GET_PROFILES and GET_PROFILE_SEARCH both relate to the user profile model.  Calling either of these constants (which connect to actions), will update the state of profile.

Always return state as a default. 

Each reducer is listed in the main reducer.


##**profileReducer**:
``` [javascript]
import {
    GET_PROFILES,
    GET_PROFILE_SEARCH
} from '../constants/apiConstants';

const profileReducer = (state = {}, action) => {
    const {
        endpoint,
    } = action;
    switch (endpoint) {
    case GET_PROFILES:
        return action.response;
    case GET_PROFILE_SEARCH:
        return {};
    default:
        return state;
    }
};
export default profileReducer;
```


##**hobbyReducer**:
``` [javascript]
import {
    GET_HOBBIES,
} from '../constants/apiConstants';

const hobbyReducer = (state = {}, action) => {
    const {
        endpoint,
    } = action;
    switch (endpoint) {
    case GET_HOBBIES:
        return action.response;
    default:
        return state;
    }
};
export default hobbyReducer;
```


##**Components**:


Everything seen happens in the component.


##**HobbyList Component Imports**:

Start a component with imports.
Most importantly, import 'React'.

'connect' from react-redux connects the various aspects of the page (the HobbyDashboard html class, the mapStateToDispatch, and mapStateToProps).

'map' is a way to loop through the profiles of all people.

From the actions imports pull in the needed functions.


``` [javascript]
import React from 'react';
import { connect } from 'react-redux';
import { map } from 'ramda';

import {
    getListApplicants,
    getApplicantSearch,
    getAllUsers,
    getUsersSearch,
    getApplicantCourseFilter,
    getCourses,
} from '../actions/apiActions';
```


##**HobbyList Component props and constants**

Start the react component by naming the class.

Next, define the static propTypes.  These include any states used, ComponentDidMount, and any functions used in the component not needed for actions.  State propTypes are '.object'.  Functions are '.func'.

Connect each propType to the props, most will be part of the constant this.props.

Define additional constants.  

```[javascript]
class HobbyDashboard extends React.Component {
    static propTypes = {
        profile: React.PropTypes.object,
        hobbies: React.PropTypes.object,
        onComponentDidMount: React.PropTypes.func,
        onHobbySearch: React.PropTypes.func,
    };

    componentDidMount() {
        this.props.onComponentDidMount();
    }

    render() {
        const {
            profile,
            hobbies,
            onHobbySearch,
        } = this.props;

        const all_profiles = profile.results ? profile.results : [];

        const profileList = all_profiles ? (map((person) => {
            return (
                <tbody key={person.id} >
                    {person.hobby ? (
                        <tr>
                            <td>
                                {person.hobby.name}
                            </td>
                            <td>
                                {person.hobby.category}
                            </td>
                            <td>
                                {person.city}
                            </td>
                            <td>
                                {person.state}
                            </td>
                        </tr>
                    ) : null
                    }
                </tbody>
            );
        })(all_profiles)) : [];

        const hobby_categories = hobbies.results ? (map((hobby) => {
            return (
                <option key={hobby.id} value={hobby.id}>
                    {hobby.category}
                </option>
            );
        })(hobbies.results)) : null;
    ```

    ```[javascript]
        return (
            <div className="container-fluid">
                <h1>Hobby Connection Board</h1>
                <form onSubmit={onHobbySearch('query')} className="row">
                    <fieldset className="col-sm-6">
                        <input
                          type="text"
                          className="form-control"
                          placeholder="SEARCH (Hobby, Category, City, State)"
                          onChange={onHobbySearch('query')}
                          autoFocus
                        />
                        <label htmlFor="query" className="form-label" />
                        <label htmlFor="category" className="form-label col-sm-4" id="category">Filter by Hobby Categories</label>
                        <select className="form-control" id="category" onChange={onHobbySearch('category')} >
                            {hobby_categories}
                        </select>
                    </fieldset>
                    <button className="btn btn-primary col-sm-2">Search</button>
                </form>
                <div className="row">
                    <div className="table-responsive">
                        <table className="table table-striped table-hover">
                            <thead>
                                <tr>
                                    <th>Hobby</th>
                                    <th>Category</th>
                                    <th>City</th>
                                    <th>State</th>
                                </tr>
                            </thead>
                            {profileList}
                        </table>
                    </div>
                </div>
            </div>
        );
    }
}
```

```[javascript]
function mapStateToProps(state) {
    return {
        profile: state.profile,
        hobbies: state.hobbies,
    };
}

function mapDispatchToProps(dispatch) {
    return {
        onComponentDidMount() {
            dispatch(getProfiles());
            dispatch(getHobbies());
        },
        onHobbySearch(field) {
            return (e, ...args) => {
                const value = typeof e.target.value !== 'undefined' ? e.target.value : args[1];
                const update = {
                    [field]: value,
                };
                dispatch(getProfileHobbySearch());
            };
        },
    };
}

export default connect(mapStateToProps, mapDispatchToProps)(HobbyDashboard);
```

The functions from apiActions are then called inside the Component mapStateToDispatch.

The calls and search functions initiated in the React Redux frontend then need the Django backend to filter the results and return a new set of profiles to the profile state, through the GET_PROFILE_SEARCH, on the profile reducer (also the type on the getProfileHobbySearch apiAction function).


##**Viewsets**:
``` [python]
from rest_framework import viewsets, generics
from django_filters.rest_framework import OrderingFilter

from django.contrib.auth.models import User
from .models import *
from .serializers import *

class HobbyViewSet(viewsets.ModelViewSet):
    queryset = Hobby.objects.all()
    serializer_class = HobbySerializer

class ProfileViewSet(viewsets.ModelViewSet):
    queryset = Profile.objects.all()
    serializer_class = ProfileSerializer

    def get_queryset(self):
        queryset = Profile.objects.order_by('state')
        state = self.request.query_params.get('state', None)

        query_text = self.request.query_params.get('query', None)
        if query_text is not None:
            query_text = query_text.lower()
            tmp_set = queryset.filter(city__icontains=query_text)
            tmp_set |= queryset.filter(state__icontains=query_text)
            tmp_set |= queryset.filter(zipcode__icontains=query_text)
            tmp_set |= queryset.filter(hobby__name__icontains=query_text)
            tmp_set |= queryset.filter(hobby__category__icontains=query_text)

            queryset = tmp_set

        return queryset.order_by('state')

class AllUsersViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = CurrentUserSerializer
```

The viewsets filter all profiles on the backend based on the query entered in the React search.