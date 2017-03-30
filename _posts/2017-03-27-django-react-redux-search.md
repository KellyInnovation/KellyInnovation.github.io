---
layout: post
title: Django React Redux Searches
---
The goal is to create a hobby board that can connect people with similar hobbies in a given area.

This will be with a Django backend and a React / Redux frontend.

In order to be useful, tables should be searchable by hobby, category, city, or state.


**Model**:

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
    user_hobby = models.ForeignKey(Profile, verbose_name='profile', blank=True, null=True, related_name='hobbies')
    name = models.CharField('hobby name', max_length=200, null=True, blank=True)
    category = models.CharField('hobby category', max_length=200, null=True, blank=True)
```

Hobby has a ForeignKey to Profile to connect the two and allow for multiple hobbies to connect with one person.

Create Serializers to connect to the frontend.

**Serializer**: 
``` [python]
from django.contrib.auth.models import User
from rest_framework import serializers

from .models import *

class HobbySerializer(serializers.ModelSerializer):

    class Meta:
        model = Hobby
        fields = (
            'id',
            'user_hobby',
            'name',
            'category',
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

The HobbyField creates a way to access information about hobbies from the profile state.  For example, on the front end, profile.hobby.name will give the name of a hobby associated with that profile without additional API calls.

In order to connect the Django backend to the React / Redux front end, create urls for API.

**URLs for API**:
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

The frontend API calls use profile and hobby as part of the url designations to indicate which serializers to utilize.

These API calls are made in actions (the API_CALL constant functionality was defined in middleware - not shown).

**apiActions**:
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

Constants (type in all caps) are listed in constants.

**apiConstants**:
``` [javascript]
export const BASE_URL = '/api';

export const GET_PROFILES = 'GET_PROFILES';
export const GET_HOBBIES = 'GET_HOBBIES';
export const GET_PROFILE_SEARCH = 'GET_PROFILE_SEARCH';
```

The functions from apiActions are then called inside the Component mapStateToDispatch.

**HobbyList Component**:
``` [javascript]
import React from 'react';
import { connect } from 'react-redux';
import { Link } from 'react-router';
import { map } from 'ramda';

import {
    getListApplicants,
    getApplicantSearch,
    getAllUsers,
    getUsersSearch,
    getApplicantCourseFilter,
    getCourses,
} from '../actions/apiActions';

import {
    formUpdate,
} from '../actions/formActions';

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
                dispatch(formUpdate(update));
                dispatch(getProfileHobbySearch());
            };
        },
    };
}

export default connect(mapStateToProps, mapDispatchToProps)(HobbyDashboard);
```

The profile and hobbies states seen in the Component come from the reducers.

**profileReducer**:
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

**hobbyReducer**:
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

Each reducer is listed in the main reducer.

The calls and search functions initiated in the React Redux frontend then need the Django backend to filter the results and return a new set of profiles to the profile state, through the GET_PROFILE_SEARCH, on the profile reducer (also the type on the getProfileHobbySearch apiAction function).


**Viewsets**:
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