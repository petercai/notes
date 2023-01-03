# Configure Java Util Logging (JUL) to use a more production/docker ready pattern using SimpleFormatter
Java Util Logging (JUL) is often criticized but it is a solid logging framework and, even if you will probably read the opposite, it is as fast as any fashion framework (as soon as you tune the handlers). Its main drawback is to be globally configured in several places (a lot of the config comes from the JVM when not integrated specifically to your environment like Apache Tomcat does) but its big advantages is that it is the default impl for a lot of Java libraries and it can easily unify logging in an application (using slf4j-jdk14 or so).

In docker erea, you will likely use the _ConsoleHandler_ which output the logs on stderr by default (with very few efforts you can log on stdout but it does not help much in terms of deployments in general) and using _SimpleFormatter_ default format, the output will likely look like:

```null
sept. 28, 2020 4:35:04 PM org.apache.karaf.main.Main launch
INFOS: All initial bundles installed and set to start
```

One issue with this output is that it uses two lines (so it is not that nice to grok to send the log events to a log aggregator like kibana) and that the date is hard to parse. It would be so nice if it could be:

```null
2020-09-28T14:28:50Z [INFO] [org.apache.karaf.main.Main launch] All initial bundles installed and set to start
```

This is exactly the same content but:

1.  It uses a single line
2.  Log level, source (class + method) are wrapped in square brackets (so trivial to parse)
3.  Order of the fields is a bit different

To achieve this output, you only have to know that _SimpleFormatter_ is configurable through a system property called _java.util.logging.SimpleFormatter.format_. It uses _String.format_ - or _java.util.Formatter_ - syntax.

Previous line was output using this configuration:

```null
'-Djava.util.logging.SimpleFormatter.format=%1$tFT%1$tTZ [%4$s] [%2$s] %5$s%6$s%n'
```

Note: the simple quotes are there to avoid UNIx interpolation, depending where you configure that, you can omit them.

This configuration hardcoded the timezone but if you replace the two first patterns by a _toString()_ pattern you will get the direct _ZonedDateTime_ output containing the right timezone:

```null
'-Djava.util.logging.SimpleFormatter.format=%1$s [%4$s] [%2$s] %5$s%6$s%n'
```

With this line the log now looks like:

```null
2020-09-28T14:28:50Z[GMT] [INFO] [org.apache.karaf.main.Main launch] All initial bundles installed and set to start
```

Note: if the date offset is the same as the zone, the _\[GMT\]_ string will not appear. This is configurable on the JVM too.

In the same category: