
<!---
    ::::
    ::  Author: Bryan McCoy
    ::  Title: Who moved my Web.Config?
    ::  Date: 1/20/2019
    ::  Tags: How to, Configuration, .Net Core
    ::  Live: Yes
    ::::
--->
## Who moved my Web.Config?

So good news, my site is up.  Bad news, I am only able to pull blog entries from a single location.  The address of the blog entries are literally hard coded in a class file.  This is not good!  I need the address of the blogs to be loaded from some sort of configuration file.  It would also be cool if we had different configurations for different environments.  So lets add that, after all this site is a work in progress!

So, if you're coming from a pre .Net Core era you are all to familiar with the infamous Web.Config or App.Config.  If you were really cool you had no less than 10 versions of Web.[some obscure environment].config and some magic tool that "transformed" your Web.Config into something you hopefully wanted.  

In .Net Core there are similar methods.  We now have appsettings.json, and a ton more options.  Including the option to create your own whatever.json settings file.  Today we are going to keep it simple.  We will be looking at appsettings.json.  We will use this file to help us control where we pull blogs from depending on the ASPNETCORE_ENVIRONMENT environment variable (more on this later). 
<!--- End Preview --->

## THE PROBLEM
---
Well, here it is… Do you see it?
``` c#
public class GitHubClient : IGitHubClient
{
    private HttpClient _httpClient;
    private string _url;

    public GitHubClient(IHttpClientFactory httpClientFactory)
    {
        _httpClient = httpClientFactory.CreateClient();
        _url = "https://api.github.com/repos/bmccoy04/CoreBlogger/contents/BlogEntries/"; 
        _httpClient.DefaultRequestHeaders.Add("User-Agent", "request");
    }
        public async Task<T> GetEntriesAsync<T>()
    {
        var content = await _httpClient.GetStringAsync(_url);                
        return JsonConvert.DeserializeObject<T>(content);
    }
    ...
}
```

.Net Core gives us the all new IHttpClientFactory,  This is NOT the best way to use it! We’ll fix that in a later blog post.  What we have here is a client getting injected, so that’s good.  We then use the factory to create a client, seems simple.  Then we have this hard coded url string being put into a local variable... That’s no good.  We are also setting some agent headers for the client, then later on we can see where the call is made and we stick the results into an object.

So the hardcoded url... If we ever wanted to change this we would literally have to change the code and re-build and possibly re-deploy it.  For local development that is fine, I guess, but what if we have multiple testing/production environments.  Gets a little harder to automate those deployments.


## ENTER ICONFIGURATION
---
.Net Core really embraces dependency injection.  It also provides us with a service we can inject in to help us get configuration values.  Our app is actually configured to use the appsetting.json files, and IConfiguration service already.  This is provided to us in the WebHost.CreateDefaultBuilder method in Program.cs.  So all we have to do is enter our value in the appsettings.json file and inject the configuration service.

Appsettings.json
``` json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "BlogEntriesUri": "https://api.github.com/repos/bmccoy04/CoreBlogger/contents/BlogEntries/"
}
```

GitHubClient.cs

``` c#
public GitHubClient(IHttpClientFactory httpClientFactory, ILogger<GitHubClient> logger, IConfiguration configuration)
{
    _httpClient = httpClientFactory.CreateClient();
    _url = configuration.GetValue<string>("BlogEntriesUri" );
    _httpClient.DefaultRequestHeaders.Add("User-Agent", "request");
    
    logger.LogInformation( "----------- Here -----------------");
    logger.LogInformation(configuration.GetValue<string>("BlogEntriesUri"));
}
```


As you can see.  We just need to tell the configuration service what the key is that we set in our appsettings.json file is.  Then the service will go and find the value for us.  Sounds familiar?  Works just like our web configs of the past!

For fun, and my prefered way of debugging, I injected ILogger<GitHubClient> to log the value of our BlogEntriesUri setting. As you can see below, everything worked as expected

![Console Output](https://i.imgur.com/8B0jhCd.pnghttps://imgur.com/8B0jhCd)


## DIFFERENT ENVIRONMENTS
---
So now that we can access our values in our settings file, we need to “transform” it per our environment.  App settings files do act similar to Web.Configs in that you can have appsettings.development.json.

What will actually happen here is when your app is ran your app will use the ASPNETCORE_ENVIRONMENT variable to determine which appsettings.json files to use.  I say files because your app will actually take the base file and environment specific file and “put” them together.  Here we can tell that my app is running with the “Development” environment variable. 

![Console Output](https://i.imgur.com/bH77H03.png)

So if I add my setting to my appsettings.development.json, I should get my desired results.

So my two files are as below.

Appsettings.json

![Appsettings.json](https://i.imgur.com/3uHFmFj.png)

Appsettings.development.json

![Appsettings.Development.json](https://i.imgur.com/2PPGkdd.png)

And now when we run, we should see our URI and not “Not what we want!”.  Because our app will take whatever is in our environment config and replace what is in our default config.

![Console after appsettings.Development.json](https://i.imgur.com/5BluWR1.png)

Now we can add a 3rd, 4th, 5th, etc. settings file to our project for the different environments we want to deploy to/work in.  Here we just talked about local development using the development environment.  When we deploy to our production server our app will run with the production environment variable.  So we will have to set up another appsettings file for that environment.  Below you can see the files included in “Release” build of our application.  

![Added Production appsettings config](https://i.imgur.com/qyjxlPU.png)

Well, there you have it.  Short and simple. Latter we will talk about how we can create our own settings files by using the ConfigurationBuilder class.  For now thought this will get us what we need.  I will set up my production appsettings file to point to my production blog entries repository for my blog entries and set up my development appsettings file to point to my test blog entries repository.

Please be sure to stay tuned for future blog entries!
