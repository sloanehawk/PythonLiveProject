# Python Live Project
## Introduction
For two separate two-week sprints at the tech academy, I worked on a team with other software development students to create an interactive hobby management website using Python, Django, SQLite, and HTML/CSS. The software we created was designed to help hobbyists keep track of their collections of items, as well as interact with APIs and use DataScraping to get up to date information on things related to those collections and hobbies. During the first two-week sprint I created my basic app, which includes a database collection manager for craft beers and CRUD functionality. In the second sprint, I created a Restful API interface that allows users to select a brewery and get an up-to-date list of all the beers for that brewery with related information. I also added a news page that uses Beautiful Soup and Data Scraping to pull the latest news articles from https://www.brewbound.com/. Below are descriptions of the stories I worked on along with code snippets and screenshots.
## Stories
* [Build the Basic App](#build-the-basic-app)
* [Create a Model](#create-a-model)
* [Index Page](#index-page)
* [Details Page](#details-page)
* [Edit and Delete Functions](#edit-and-delete-functions)
* [API Service](#api-service)
* [Data Scraping with Beautiful Soup](#data-scraping-with-beautiful-soup)



### Build the Basic App
My first task was to build the basic app for my craft beer tracking service and get the home page to render. I did this by running ‘python manage.py startapp CraftBeer’, and registering the app within the main project settings.py file. I then created the base and home templates with basic content, and added a function to views.py to render the home page. Finally, I registered my urls within the MainApp and then created urls.py for my app and home page.
The resulting home page looked like this:




### Create a Model
My task for this story was to create a model for the collection item I’m tracking (beer) and add the ability to create a new item. 
I first added a Beer class to models.py:

    class Beer(models.Model):
       beer_name = models.CharField(max_length=75)
       brewery = models.CharField(max_length=50)
       style = models.CharField(max_length=20)
       abv = models.CharField(max_length=5)
       avg_score = models.DecimalField(decimal_places=10,max_digits=12)

I then created the model form in forms.py, which includes any inputs the user has to make. I chose to include all fields from the Beer model:

    class BeerForm(ModelForm):
       class Meta:
           model = Beer
           fields = '__all__'

I added a template for creating a new item:

    {% block templatecontent %}
    <section>
       <div class="flex-container">
           <form method="post">
               {% csrf_token %}
               <table>
                   {{ form.as_table }} <!-- this only renders the td tags, still need the table tags -->
               </table>
               {{ form.non_field_errors }}
               <button class="primary-bright-button" type="submit"> Add to Collection</button>
           </form>
       </div>
    </section>
    {% endblock %}

Then I created a function in views.py that renders the create page and uses the model form to save the item to the database:

    def add_beer(request):
       form = BeerForm(request.POST or None)  # Gets the posted form, if one exists
       if form.is_valid():  # Checks the form for errors, to make sure it's filled in
           form.save()  # Saves the valid form/beer to the database
           return redirect('listBeers')  # Redirects to the index page
       else:
           print(form.errors)  # Prints any errors for the posted form to the terminal
           form = BeerForm()  # Creates a new blank form
       return render(request, 'CraftBeer/craftbeer_create.html', {'form': form})

I added the path to urls.py:

    path('AddToCollection/', views.add_beer, name='createBeer')

### Index Page
The goal for this story was to display the information from the database to an index page. 
I accomplished this by adding a function in views.py that gets all the beers from the database and sends them to the template:

    def index(request):
       get_beers = Beer.beers.all()  # Gets all the current beers from the database
       context = {'beers': get_beers}  # Creates a dictionary object of all the beers for the template
       return render(request, 'CraftBeer/craftbeer_index.html', context)

Then I created the template to display each beer from the database with each field for that beer:

    <table class="table-striped">
       <tr>
           <th class="col-md">Beer Name</th>
           <th class="col-md">Brewery</th>
           <th class="col-md">Style</th>
           <th class="col-md">ABV</th>
           <th class="col-md">Average Score</th>
       </tr>
       {% for beer in beers %}     <!-- creates a new row for each beer in the collection -->
       <tr>
           <td class="col-md">{{beer.beer_name}}</td>
           <td class="col-md">{{beer.brewery}}</td>
           <td class="col-md">{{beer.style}}</td>
           <td class="col-md">{{beer.abv}}</td>
           <td class="col-md">{{beer.avg_score}}</td>
       </tr>
       {% endfor %}
    </table>

Then I added the path to urls.py:

    path('Collection/', views.index, name='listBeers')

### Details Page 
For this story, my task was to create a details page that shows the details of any single beer from the database, and link this to the index page for each beer.

I added the following code to views.py:

    def details_beer(request, pk):
       beer = get_object_or_404(Beer, pk=pk) # get beer record if it exists
       return render(request, 'CraftBeer/craftbeer_details.html', context={'beer': beer})

Then I added a new template for the details page which included this table:

    <table>
       <tr>
           <th>Beer Name</th>
           <td>{{beer.beer_name}}</td>
       </tr>
       <tr>
           <th>Brewery</th>
           <td>{{beer.brewery}}</td>
       </tr>
       <tr>
           <th>Style</th>
           <td>{{beer.style}}</td>
       </tr>
       <tr>
           <th>ABV</th>
           <td>{{beer.abv}}</td>
       </tr>
       <tr>
           <th>Average Rating</th>
           <td>{{beer.avg_score}}</td>
       </tr>
    </table>

I added the path to urls.py:

    path('Collection/<int:pk>/Details/', views.details_beer, name='beerDetails')

Finally, I added another column to the index page with a button for each beer to view details of that specific beer: 

    <td class="col-md"><button class="primary-bright-button" type="button" onclick=" location.href='{% url 'beerDetails' beer.pk %}'">Details</button></td>

### Edit and Delete Functions
The purpose of this story was to add edit and delete functionality to the details page for each beer. 

This required adding two functions to views.py.
The edit function:

    def edit_beer(request, pk):
       beer = get_object_or_404(Beer, pk=pk) # get beer record if it exists
       if request.method == "POST":
           form = BeerForm(request.POST, instance=beer)  # gets the posted form
           if form.is_valid():  # Checks the form for errors, to make sure it's filled in
               beer = form.save()  # Saves the valid form/beer to the database
               beer.save()
               return redirect('beerDetails', pk=beer.pk)
       else:
           form = BeerForm(instance=beer)
       return render(request, 'CraftBeer/craftbeer_edit.html', {'form': form})

The delete function:

    def delete_beer(request, pk):
       beer = get_object_or_404(Beer, pk=pk)
       if request.method == "POST":
           beer.delete()
           return redirect('listBeers') # redirects to the index page
       return render(request, "CraftBeer/craftbeer_confirmdelete.html", context={'beer': beer})

I added the edit and delete buttons to the details page:

    <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'editBeer' beer.pk %}'">Edit
    </button>
    <button class="primary-bright-button" type="button" onclick="location.href='{% url 'deleteBeer' beer.pk %}'">Delete
    </button>

Two new templates needed to be created for the edit and delete pages.

The form included in the edit template:

    <form method="post">
       {% csrf_token %}
       <table>
           {{ form.as_table }}
       </table>
       {{ form.non_field_errors }}
       <button class="primary-bright-button" type="submit">Save</button>
    </form>

The form included in the delete template:

    <form method="post">
       {% csrf_token %}
       <h1>Are you sure you want to delete {{beer.beer_name}}</h1>
       <button class="primary-bright-button" type="submit">Delete</button>
       <button class="primary-bright-button" type="button" onclick="location.href='{% url 'listBeers' %}'">Cancel
    </button>

Finally, I registered the paths in urls.py:

    path('Collection/<int:pk>/Edit/', views.edit_beer, name='editBeer'), # edit details for a beer
    path('Collection/<int:pk>/Delete/', views.delete_beer, name='deleteBeer'), # delete beer

### API Service 
For my craft beer API service, I wanted users to be able to select a brewery from the database, and receive an up-to-date list of beers for that brewery with some basic information. BreweryDB.com is the API I chose to use.

In a new python file for my api service I added two functions.

A function that sends a get request to the API endpoint to get all breweries from the database to populate the dropdown:

    def get_breweries():
       parameters = {'format': 'json', 'key': API_KEY}
       response = requests.get(API_ENDPOINT + "/breweries", params=parameters)
       response = response.json()
       brewery_dict = response['data']     #dictionary of brewery dictionaries
       return brewery_dict

And a function that sends a get request to get all the beers for the selected brewery:

    def get_beers(brewery_id):
       parameters = {'format': 'json','key': API_KEY}
       response = requests.get(API_ENDPOINT + '/brewery/' + brewery_id + '/beers', params=parameters)
       response = response.json()
       beer_dict = response['data']
       return beer_dict


In views.py I added two views.

A view function to render the main API page:

    def api_response(request):
       context = {'breweries': get_breweries()}        #Creates a dictionary item of all the breweries
       if request.method == 'POST':                #This block will handle all post backs from the forms
           print(request.POST)  #This was used for debugging purposes
           brewery_id = request.POST['brewery']  #Gets the value of brewery from the dropdown menu, an element of the Post dictionary
           matches_page = '../{}/Beers'.format(brewery_id)    #Dynamically sets the redirect page
           return redirect(matches_page)       #Sends user to matches page, url contains the code the view function needs

       return render(request, 'CraftBeer/craftbeer_api.html', context)

And a view function to parse through the JSON response and render the results page, :

    def beers(request, brewery_id):
       beer_dictionary = get_beers(brewery_id)       #Gets the Json response dictionary of beers for that brewery
       context = {'beers': []}  # add empty dict with 'beers' key
       for item in beer_dictionary:  #Iterates through the matches within the JSON response
           # assign each value to beer object if it exists in the JSON response (default value is None)
           beer = {'beer_id': item.get('id'),
                   'beer_name': item.get('name'),
                   'beer_abv': item.get('abv'),
                   'beer_style': item.get('style', {}).get('name'),
                   'description': item.get('style', {}).get('description'),
                   'img': item.get('labels', {}).get('large')}

           print(beer)                                     #Used for debugging, prints the beer object to the terminal
           context['beers'].append(beer)                 #Adds the beer item to the array in the dictionary, then iterates through next item

       return render(request, 'CraftBeer/craftbeer_beers.html', context) #Sends completed dictionary of beers to template

I added two new templates.

One for the main API page with a dropdown populated with all the breweries from the database::

    <form method="post">
       {% csrf_token %} <!-- Django needs a csrf token for every form -->
           <label>Select a Brewery:</label>
           <select name="brewery" id="brewery">
               {% for brewery in breweries %}
                   <option value="{{brewery.id}}">{{brewery.name}}</option>
               {% endfor %}
           </select>
       <button class="primary-bright-button" type="submit" name="submit" value="submit">Get All Beers</button>
    </form>

And one for the results page, which displays the beer name, ABV, style, logo, and description :

    <table class="beer_table">
       <tr>
           <th>Beer</th>
           <th>ABV</th>
           <th>Style</th>
           <th colspan="2">Logo</th>
           <th colspan="7">Description</th>
       </tr>
       {% for beer in beers %}
           <tr>
               <td>
                   {{beer.beer_name}}
               </td>
               <td>
                   {{beer.beer_abv}}%
               </td>
               <td>
                   {{beer.beer_style}}
               </td>
               <td colspan="2">
                   <div class="brewery_img">
                       <img src="{{beer.img}}">
                   </div>
               </td>
               <td colspan="7">
                   {{beer.description}}
               </td>

           </tr>
       {% endfor %}
    </table>

And finally I registered the paths in urls.py:

    path('ApiService/', views.api_response, name='craftbeerApi'),           #main page for API service
    path('<str:brewery_id>/Beers/', views.beers, name='craftbeerMatches'),  # specific page for beer matches

Here is part of the API results page for Laughing Dog Brewery:



### Data Scraping with Beautiful Soup
I chose to scrape data from https://www.brewbound.com/category/news using Beautiful Soup. 

Here is the function for data scraping that I added to views.py which includes the get request, and parsing through the HTML:

    def brewbound_news(request):
       page = requests.get("https://www.brewbound.com/category/news")     #Get https://www.brewbound.com/category/news as an html document
       print(page.status_code)                   #Used for debugging to ensure a 'success' code of 200
       soup = BeautifulSoup(page.content,'html.parser') #Initial processing of the html by beautiful soup, soup is now a navigatable object
       # print(soup.prettify())
       nodes = soup.find_all(class_='post')            # find all article tag instances on the page
       articles = []                               #Create blank array to add articles to
       for node in nodes:                          #Iterates through all the article objects
           title = node.find_all('a')[-1].get_text()       #Sets title equal to the text of the a tag, indexing to get a tag for article instead of image
           link = node.find_all('a')[-1].get('href')       #Sets link equal to the href of the a tag
           timestamp = node.find_all("time")[-1].get_text()   #Uses class and sibling relationship to get the timestamp
           article={'title': title, 'url': link, 'date': timestamp}  #Creates and article object dictionary with needed elements
           articles.append(article)            #Adds article dictionary item to the array before iterating through next node
           print(article)  #for debugging
       context = {'articles': articles}          #Creates a dictionary element for the articles to pass to the template
       return render(request, 'CraftBeer/craftbeer_news.html', context)


I created a new template to display the latest news articles that were scraped from the Brewbound website:

    <div class="flex-container wrapper news">
       {% for article in articles%} <!-- Iterates through the array of articles -->
           <!-- Creates a new line for each item, sets the href to the full url, gives the article title and date -->
           <p><a href="{{article.url}}">{{article.title}}</a> <span>{{article.date}}</span></p>
       {% endfor %}
    </div>

And lastly I registered the url path in urls.py.
