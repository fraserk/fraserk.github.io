---
layout: post
title: "Laravel and Vuejs Pagination"
date: 2017-01-03 -0500
categories: Laravel,VueJS
image: "/assets/images/vuejspagination.png"
---

After playing around with Vuejs and Laravel I decided to write about creating Pagination with Laravel as the backend and VueJS as the frontend.

## Tools Used:
[Laravel][f5e2ec21]
[VueJS][23ac184e]
[Get the code on Github][708c965d]

  [708c965d]: https://github.com/fraserk/vuejspagination "Source"

### The Set up.
Going to first use laravel to create some fake data and set-up the routes.  I will assume you know how to set-up a new Laravel site.

Lets create a model factory to get some test data in the database. This is going to be really simple.
All we going to use for the data is the contact name.
~~~
$factory->define(App\Contact::class, function (Faker\Generator $faker) {
    return [
        'name' => $faker->name
    ];
});
~~~

### Create the seeder file.
~~~
php artisan make:seeder ContactSeeder
~~~
_That will create a ContactSeeder.php file in database/seeds_

### Now lets create the model and migration

Run the following Laravel Artisan Command.

~~~
php artisan make:model Contact -m
~~~
_That will create a Contact model and migration file._

Edit the migration file

Just add a name field as a string
~~~
public function up()
   {
       Schema::create('contacts', function (Blueprint $table) {
           $table->increments('id');
           $table->string('name');
           $table->timestamps();
       });
   }
~~~

#### Make sure you're connected to a database.  Edit the .env file with you database info.
Run the migration file
~~~
php artisan migrate
~~~


### Now we just have to edit out seed file and run it.

Open database/seed/ContactSeeder.php

Add the following to the run function

~~~
public function run()
   {
       factory(App\Contact::class, 50)->create();
   }
~~~
_That will create 50 fake names in the database we can use._

One last step for the database part.

Open database/seeds/DatabaseSeeder.php

Add a call to the ContactSeeder we created earlier.
~~~
 $this->call(ContactSeeder::class);
~~~
Then Run the Laravel Artisan Command
~~~
php artisan db:seed
~~~
__The database should now have 50 records of names in it.__

## Now lets create the routes and controller

Create 2 simple routes to access the data.
~~~
Route::get('/api/contacts', 'ContactController@show');
~~~
This route will be use to get the data in VueJS.

In the ContactControllerController show method, let just retrieve all the data and paginate it.
Laravel will automatically return the data as json.

Create the following method.

~~~
public function show()
    {
        return Contact::paginate(10);
    }
~~~

If you navigate to localhost/api/contacts in the browser, you should see you data from the database.

![Laravel Json Pagination](/assets/images/jsondata.png)

## VueJS time

_Im going to assume you know how to setup VueJS with laravel._


We are going to use the Example.vue component that come with Laravel.
in recources/assets/js/components

For the sake of keeping this simple we are just going to use the welcome view that comes with laravel.  Edit it like the following.
_Now edit the examle.vue file._

~~~
@extends('layouts.app')
  @section('content')

          <example>

          </example>

  @endsection
~~~

Here is what the table in the template looks like.

~~~
<template>
<table class="table table-striped">
                      <thead>
                        <tr>
                          <td>ID</td>
                          <td>NAME</td>
                          <td>Created At</td>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="contact in contacts.data">
                          <td>{{contact.id}}</td>
                          <td>{{contact.name}}</td>
                          <td>{{contact.created_at}}</td>
                        </tr>

                          <tr>
                            <td colspan="5">
                              <nav araia-label="Page navigation">
                                <ul class="pagination">
                                  <li v-show="contacts['prev_page_url']">
                                    <a href="#" @click.prevent="getPreviousPage">
                                    <span>
                                      <span aria-hidden="true">&laquo;</span>
                                    </span>
                                    </a>
                                  </li>
                                  <li v-for="n in contacts['last_page']">
                                    <a href="#" @click.prevent="getPage(n)">
                                    <span >
                                      {{n}}
                                    </span>
                                    </a>
                                  </li>
                                  <li  v-show="contacts['next_page_url']">
                                    <a href="#" @click.prevent="getNextPage">
                                    <span>
                                      <span aria-hidden="true">&raquo;</span>
                                    </span>
                                    </a>
                                  </li>
                                </ul>
                              </nav>



                            </td>
                          </tr>

                      </tbody>
                    </table>
</template>
~~~
Here is the vue methods looks like, with the ajax calls to  to laravel.

~~~
<script>
    export default {
      data(){
        return {
          contacts: {}
        }
      },
        mounted() {
          this.getContactList();
        },
        methods: {
          getContactList(){
            this.$http.get('/api/contacts').then((response)=>{
              this.$set(this.$data, 'contacts',response.data);
            }, (response)=>{
              console.log(response.data)
            });
          },
          getPage(page){
            this.$http.get('/api/contacts?page='+page).then((response)=>{
              this.$set(this.$data, 'contacts',response.data);
            },(response)=>{
            });
          },
          getPreviousPage(){
            this.$http.get(this.contacts['prev_page_url']).then((response)=>{
                this.$set(this.$data, 'contacts',response.data);
            },(response)=>{
            });
          },
          getNextPage(){
            this.$http.get(this.contacts['next_page_url']).then((response)=>{
                this.$set(this.$data, 'contacts',response.data);
            },(response)=>{
            });
          },
        }
    }
</script>
~~~

Here's a quick demo.

![Pagination demo](/assets/images/pagination-demo.gif)






  [f5e2ec21]: https://laravel.com "Laravel"
  [23ac184e]: vuejs.org "VueJS"
