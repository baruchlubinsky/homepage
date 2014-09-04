+++
Categories = ["Development", "Golang"]
Description = "Some notes about building this website."
Tags = ["development", "golang", "tutorial"]
Title = "This website"
Section = "blog"
Date = 2014-08-12T10:18:56Z
+++

I built this website using [Hugo](http://hugo.spf13.com/). I found it a little tricky to get started, but it is very new software. Now that it is up and running, I'm really getting into it. 

The styling use the [Hyde theme](https://github.com/spf13/hyde) for Hugo which is based on MDO's [package](https://github.com/poole/hyde) of the same name.

Once I've tidied things up a bit, I'll post the source code on Github.

The website is hosted on S3. Uploading is simple with [S3cmd](http://s3tools.org):
{{% highlight bash %}}
s3cmd sync -P --exclude="DS_Store" public/ s3://baruch.lubinsky.co.za
{{% /highlight %}}