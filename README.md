# django-react-native
### Create react native mobile apps with django 

### React
Why React Native? It is created by facebook and all tye facebook products and their apps uses it .

### Django
Django is the best python web framework . It powers instragram , mozzila , natgeo, pinterest, dropbox, disqus, sentry etc .

### API
We will release API with the django rest framework . The response is gonna be in JSON format so the users will get The modified css classed data . We will grab JSON data via react native , then we will add css designs and make it a full featured app. So we will create a Hostel blog or travel blog . The models.py is down below .

```python
from django.db import models
from django.contrib.auth.models import User


class Restaurant(models.Model):
    name = models.CharField(max_length=300)
    address = models.TextField()
    photo = models.ImageField(null=True, blank=True)
    tags = models.CharField(max_length=200)
    publish = models.DateTimeField(auto_now_add=True)
    writer = models.ForeignKey(User, default=1)

    def __str__(self):
        return self.name
    def __repr__(self):
        return repr(self.name)
```

We will then make a serializer class using Django Rest framework.

```python

from rest_framework import serializers
from food.models import Restaurant


class RestaurantSerializer(serializers.ModelSerializer):
    class Meta:
        model = Restaurant
        fields = ('id', 'name', 'photo', 'tags', 'menu', 'publish')
```

Then we create an API function in views.py which will server requests at /api . We will grab data via api.json .

```python

from django.shortcuts import render
from django.http import JsonResponse
from .models import Restaurant
from .serializers import RestaurantSerializer
from django.views.decorators.csrf import csrf_exempt


def index(request):
    rest_list = Restaurant.objects.order_by('-pub_date')
    context = {'rest_list': rest_list}
    return render(request, 'food/index.html', context)


def get_rest_list(request):
    if request.method == "GET":
        rest_list = Restaurant.objects.order_by('-pub_date')
        serializer = RestaurantSerializer(rest_list, many=True)
        return JsonResponse(serializer.data, safe=False)

```

We will be creating our React Native app now. The JSON response is like this

```json
[{
    "id": 3,
    "name": "BurgerLang",
    "address": "Dhaka",
    "photo": "burg.jpg",
    "tags": "Burger",
    "menu": "1. Cheese Burger",
    "publish": "2017-04-05T12:34:08.094007Z"
}]
```

Create React Native app, we will use CRNA (Create React Native App). 

```javascript
import React from 'react';
import { StyleSheet, Text, FlatList, ActivityIndicator, View, Image } from 'react-native';
import { List, ListItem, SearchBar, Avatar } from "react-native-elements";
import { StackNavigator } from 'react-navigation';
import { constants } from 'expo';
import HomeScreen from './src/components/home';
import DetailScreen from './src/components/detail';


export default StackNavigator({
    Home: { screen: HomeScreen,
            navigationOptions: {
                title: 'Home',
                headerBackTitle: 'Back',
            },
          },
    Detail: { screen: DetailScreen,
            navigationOptions: {
              title: 'Detail',
          },
        }
});
```

Then we will fetch the API with react . And apply some css style . 

```javascript
import React from 'react';
import { StyleSheet, Text, FlatList, ActivityIndicator, View, Image } from 'react-native';
import { List, ListItem, SearchBar, Avatar } from "react-native-elements";
import { StackNavigator } from 'react-navigation';

export default class HomeScreen extends React.Component {
 constructor(props) {
    super(props);

    this.state  = {
      loading: false,
      data: [],
      error: null,
      refreshing: false,
      base_url: "http://127.0.0.1:8000"
    }
  }

  componentDidMount() {
    this.fetchDataFromApi();

  }

  fetchDataFromApi = ()  => {
    const url = "http://127.0.0.1:8000/api.json";

    this.setState({ loading: true });

    fetch(url)
      .then(res => res.json())
      .then(res => {

        this.setState({
          data: res,
          error: null,
          loading: false,
          refreshing: false
        });
      })
      .catch(error => {
        this.setState({ error, loading : false });
      })
  };

  handleRefresh = () => {
    this.setState(
      {
        refreshing: true
      },
      () => {
        this.fetchDataFromApi();
      }
    );
  };

  renderSeparator = () => {
    return (
      <View
        style={{
          height: 1,
          width: "86%",
          backgroundColor: "#CED0CE",
          marginLeft: "14%",
          marginTop: "3%"
        }}
      />
    );
  };

  renderHeader = () => {
    return <SearchBar placeholder="Type Here..." lightTheme round />;
  };

  render() {
    return (
      <List containerStyle={{ borderTopWidth: 0, borderBottomWidth: 0 }}>
        <FlatList
          data={this.state.data}
          renderItem={({ item }) => (
            <ListItem
              onPress={() => this.props.navigation.navigate('Detail',
              {name: `${item.name}`, menu: `${item.menu}`,
              img: `${this.state.base_url}${item.photo}`,
              address: `${item.address}`})}
              avatar={<Avatar
                      source={ {uri: `${this.state.base_url} ${item.photo} `} }
                      onPress={() => console.log("Works!")}
                      containerStyle={{marginBottom: 2}}
                      avatarStyle={{resizeMode: "cover"}}
                      width={140}
                      height={130}
                />}
              title={`${item.name}`}
              titleStyle={{ fontSize: 16}}
              titleContainerStyle = {{ marginLeft: 120 }}
              subtitle={<View style={styles.subtitleView}>
            <Text style={styles.menuText}>{item.menu}</Text>
            <Text style={styles.locText}>{item.address}</Text>
            </View>}
              containerStyle={{ borderBottomWidth: 0, marginBottom: 20 }}
            />
          )}
          keyExtractor={item => item.id}
          ItemSeparatorComponent={this.renderSeparator}
          ListHeaderComponent={this.renderHeader}
          onRefresh={this.handleRefresh}
          refreshing={this.state.refreshing}

        />
      </List>
    );
  }
}

const styles = StyleSheet.create({
  // style here
});
```

## Enjoy
