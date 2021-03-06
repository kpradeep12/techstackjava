---
layout: post
title:  "Generating static pages with J2HTML"
date:   2018-09-30 12:11:10 -0500
categories: java
image: /assets/images/banners/quick-static-pages-with-J2HTML-and-Jetty.png
author: pradeep
featured: false
---

This article demonstrates the usage of J2HTML library, I recently developed this small Java class which when executed will run an embedded Jetty server and this server will provide fields, constructors and methods of any Java class or interface requested. This server acts like a Java API browser. To develop this class I used below two dependencies.

* Jetty It is a open source server.
* J2HTML It is a HTML builder library.

Below is the screenshot of the HTML page generated by J2HTML which is showing fields, constructors and methods of requested java.util.ArrayList class.

![]({{site.baseurl}}/assets/images/posts/2018/09/Jerry-server-J2HTML-response.png){: height="500px" width="650px"}{: .align-center}

This class performs below three actions;

* Create server
* Generate dynamic HTML
* Listen for the HTTP requests

Lets go through all these actions in next sections.

## Create server

Below code creates an instance of server.

{% highlight java %}
ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
context.setContextPath("/");
context.addServlet(new ServletHolder(new JavaAPI()), "/api") // <1>

Server server = new Server(4000); // <2>
server.setHandler(context);
server.start();
server.join();
{% endhighlight %}
Full class of this example can be found here JavaAPI.java.

<1> Adds a servlet 'JavaAPI' to the context.  This servlet will get executed on every request at '/api' and this will implement the logic to respond the dynamic HTML page.  
<2> Server listens on 4000 port.

## Generate dynamic HTML

J2HTML library helps in generating dynamic HTML document. Each HTML tag in the DOM will have its corresponding method in this library. Structure of the method calls resembles the structure of the document. So its easy and simple to create an HTML document with this library.

Below code snippet will give you an idea of how this library works and this code will generate a simple HTML. Look at how the method calls resembles the HTML document.

{% highlight java %}
html(
    body(
        h1("Hello, World!")
    )
).render();

//Generates below HTML
<html>
    <body>
        <h1>
            Hello, World!
        </h1>
    </body>
</html>
{% endhighlight %}
Lets go back to our Java API class. Below is the content method which uses J2HTML library to dynamic generate HTML which contains list of fields, constructors and methods of the passed class.

{% highlight java %}
private static String content(Class<?> clazz) { // <1>
        return html(
                head(
                        title(clazz.getCanonicalName()),
                        link().withRel("stylesheet")
.withHref("https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css")
                ),
                body(attrs(".container"),
                        h2(attrs(".text-center"), clazz.getCanonicalName()),
                        membersList("Fields", clazz.getDeclaredFields()),
                        membersList("Constructors", clazz.getDeclaredConstructors()),
                        membersList("Methods", clazz.getDeclaredMethods())
                )
        ).render(); // <2>
}

private static DomContent membersList(String header, Member[] members) { // <3>
        return div(
                    h5(header),
                    ul(
                        Stream.of(members)
                                .map(member -> li(member.toString()))
                                .toArray(DomContent[]::new) // <3>
                    )
        );
}
{% endhighlight %}
<1> content method needs instance of the 'Class'. 'head' and 'body' tags are passed to 'html'. J2HTML provides CreatorTag methods for each corresponding HTML tag so I am passing head and body to html. I am setting title and adding bootstrap css library in the head. Assigned 'container' class to body and showing title in the h2 tag. For each section I am calling 'membersList' method.  

<2> calling render method on html CreatorTag generates string representation of HTML document.  

<3> membersList takes header and Member[] array and generates div with header and list items.

## Listen for the HTTP requests

We need to listen for the HTTP requests and respond with dynamic HTML.

{% highlight java %}
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String clazz = request.getParameter("class"); // <1>
        response.setContentType("text/html");
        response.setStatus(HttpServletResponse.SC_OK);

        try {
            response.getWriter().println(content(Class.forName(clazz))); // <2>
        } catch (ClassNotFoundException e) {
            response.getWriter().println("Class not found: " + e.getMessage());
        }
}
{% endhighlight %}
<1> Reads the request parameter 'class'. This parameter contains the full class name.  

<2> Create 'Class' instance and pass it to content method. content method will generate HTML and returns it as string.  

Full example of this class is available on GitHubGist: JavaAPI.java When we run this class new Jetty server instance will be created and it starts and listens on 4000 port. Users can request API for any Java class or interface from the browser.

Example requests:

* http://localhost:4000/api?class=java.util.ArrayList
* http://localhost:4000/api?class=java.math.BigDecimal
* http://localhost:4000/api?class=java.util.Collection

## Conclusion

We went through code snippets of creating Jetty server instance and creating dynamic HTML using J2HTML library.