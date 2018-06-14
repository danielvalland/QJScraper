A simple, light-weight web crawler built on Jsoup and Xsoup with support for parallel processing, XPATH and predicates to gather html elements.
## Getting started ##
You can use the JAXScraper by including the jar file as a dependancy, or by downloading the source-code (zip file above). A simple web-crawl/scrape can be done in a few lines:
```java
ResultDictionary imageUrls = new ResultDictionary();
        new Scraper()
                .addExtractor(new XSoupExtractor("//img/@src"), imageUrls)
                .addDoFollow(url -> true) // follow all links
                .setMaxDepth(2) // set max-depth to 2 levels
                .run("http://vg.no"); // run scraping on http://vg.no
```
In this example, the scraper extracts all image urls all pages visited, and stores them in the imageUrls result-dictionary. A couple of special classes are used in this example: the  ResultDictionary class provides an abstract, easy way of storing results. Results are stored as mapping from origin url -> list of results from given url.  The XSoupExtractor class provides an easy way to extract elements from each page by specifying an XPATH query that will be executed on each page visited, and the result is passed to the ResultDictionary. Max depth is set to 2 levels, and the scraping is started by calling the run("url here...") method.
### Parallel processing: ###
```java
To use parallel-processing, simply use the setUseParallel() setter:
ResultDictionary imageUrls = new ResultDictionary();
        new Scraper()
                .addExtractor(new XSoupExtractor("//img/@src"), imageUrls)
                .addDoFollow(url -> true)
                .setMaxDepth(2)
		            .setUseParallel(true)
                .run("http://vg.no");
```
When set to true, each child-url (branch) from each page will be processed in parallel. Enabling parallel-processing can greatly increase speed, but does not guarantee the order in which the results will appear in the result-set. This is because, in parallel mode, several pages may be processed simountanously, and the results are added to the resultset as soon as discovered. Thus, the order will depend on the order in which pages complete/pageload of pages, which will be arbitrary.
### PageConsumers: ###
You can also handle each page visited using a PageConsumer, and interact with the JSoup elements directly:
```java
 List<Element> imgElements = new LinkedList<>(); // list (of JSoup Element) to store results
        new Scraper()
                .addPageConsumer(rootElement -> imgElements.addAll(rootElement.getElementsByTag("img")))
                .addDoFollow(url -> true)
                .setMaxDepth(2)
                .run("http://vg.no");
```
The PageConsumer is Consumer function that receives the root element of each page visited, and can be used to interact with the elements using traditional JSoup. In this example, all img tags are gathered and stored in the imgElements list.
### Error-handling: ###
To handle errors that occur while scraping, simply set an exception consumer:
```java
	new Scraper()
                .addPageConsumer(rootElement -> imgElements.addAll(rootElement.getElementsByTag("img")))
                .addDoFollow(url -> true)
                .setMaxDepth(2)
                .setExceptionConsumer((originUrl, exception) -> System.err.println("Error occured on: " + originUrl + " : "+exception.getMessage()))
                .run("http://vg.no");
```
The exception consumer receives each exception that occurs, and the url at which the exception occured.
### Following links: ###
An arbitrary amount of "link followers" can be added, and predicates are used to determine which urls to follow:
```java
	new Scraper()
                .addPageConsumer(rootElement -> imgElements.addAll(rootElement.getElementsByTag("img")))
                .addDoFollow(url -> url.contains("vg.no") && url.endsWith("/"))
                .setMaxDepth(2)
                .run("http://vg.no");
  ```
The scraper will evaluate each link-follower added, and use the predicate to determine if the url should be followed or not. In this example, the scraper follows only links that contains "vg.no" and ends with "/". Multiple "url followers" can be added as such:
```java
	new Scraper()
                .addPageConsumer(rootElement -> imgElements.addAll(rootElement.getElementsByTag("img")))
                .addDoFollow(url -> url.endsWith("/"))
	        .addDoFollow(url -> url.length() < 10)
                .setMaxDepth(2)
                .run("http://vg.no");
```
In this case, the scraper will follow all links less than 10 characters in length, AND all urls that ends with "/". The "do follow" rules are evaluated in order, such that the scraper will first follow all urls that ends with "/", and then follow all urls who's length is less than 10 characters.
