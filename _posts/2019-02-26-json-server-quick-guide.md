---
title: JSON Server quick install guide for developers who can't code  good
author: deonfourie
---

I am a Front-end developer, with a mainly graphic design background, living in a world of Back-end developers. They can do things with code in their sleep that can take me up to a whole day or more to complete. 

On many an occasion, while working on a project, I have needed to be able to get data from a Back-end that is not yet in a state that I can communicate with. This has led me to attempt to create dummy data in methods, services and anywhere else I can fit code that will hopefully help me move forward while waiting. But same problem crops up each time: It’s an awfully large amount of code that doesn’t replicate ideal conditions. No call to the Back-end is that fast and the code is a bloated mess.

Wouldn’t it be great to have my own Fake Back-end? Then I could set up everything as it should be and simply change to the correct the calls once they are made available…  Welcome to JSON SERVER


#### JSON SERVER
“A full fake REST API with zero coding in less than 30 seconds.  Created for front-end developers who need a quick back-end for prototyping and mocking.”

Or course it doesn’t tick ALL the boxes, but it does tick a fair amount. I will briefly go through the JSON SERVER installation and as well as set up a basic http service to get you going. I am assuming you already have a working solution with Node.js and are using npm to install dependencies.

1. Install globally: 
```
npm install -g json-server
```

2. Create the file “db.json” and place it pretty much where you want. I recommend the root of your application just to keep things together.

3. Populate the “database” by adding your own objects to that file. For example:
```
{
  "posts": [
    { "id": 1, "title": "json-server", "author": "typicode" }
  ],
  "comments": [
    { "id": 1, "body": "some comment", "postId": 1 }
  ]
}
```

4. Start the server: 
```
json-server --watch db.json
```
It is now watching the file you previously created. Every project you are working on can now have it’s own personal fake database. This is worth gold to a Front-end Developer..

5. Test it by going to http://localhost:3000/posts/1. You should get the following result: 
```
{ "id": 1, "title": "json-server", "author": "typicode" }
```

You can also use POST, PUT, PATCH or DELETE requests in addition to GET. There are plural and single routes available all based off of the database file you created. It can also use filter, pagination and sorting. For a full list you can visit their page here.

### Http Service Setup
Let’s set up a basic get request in your code. I will use the latest Angular2+ version for this example. I am including a few other files to flesh out your solution and get it into a good state from the start. Files will include:

config.ts: configuration info, can be extended upon but will do for this example
home.component.ts: the origin of the data call, in this case a get
data.service.ts: receives the get request, includes config file value
http-provider.service.ts: makes the final call with all the values included

app/config.ts
```
export default {
    BASE_URL: 'http://localhost:3000',
};
```

app/views/home/home.component.ts
```
import { Component, OnInit } from '@angular/core';
import { DataService } from 'src/app/services/data.service';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.scss']
})
export class HomeComponent implements OnInit {
  items: Object;
  isLoadingItems = false;

  constructor(private data: DataService) {}

  ngOnInit() {
    this.isLoadingItems = true;
    this.data.getItems().subscribe(
      data => {
        this.items = data;
        this.isLoadingItems = false;
        console.log(this.items);
      },
      err => {
        console.log(err);
        this.isLoadingItems = false;
      }
    );
  }
}
```

app/services/data.service.ts
```
import { Injectable } from "@angular/core";
import { HttpProviderService } from './http-provider.service'
import Config from '../config';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: "root"
})

export class DataService {
  constructor(private httpProvider: HttpProviderService) {}
       
  getItems() {
  	return this.httpProvider.get(`${Config.BASE_URL}/items`)
  }
}
```

app/services/http-provider.service.ts
```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class HttpProviderService {
  constructor(private http: HttpClient) {}

  get(url: string) {
    return this.http.get(url);
  }
}
```

### Conclusion
So that is how to setup JSON Server and create a basic get call to the database. The data.service.ts and http-provider.service.ts can be expanded to contain many other types of calls: post, put, delete etc. 

The End.



