1. Overview
   Logback is one of the most popular logging frameworks for Java-based applications. It has built-in support for advanced filtering, archival and removal of old log files, and sending log messages over email.

In this quick tutorial, we'll configure Logback for sending out an email notification for any application errors.

2. Setup
   Logback's email notification feature requires using a SMTPAppender. The SMTPAppender makes use of the Java Mail API, which in turn depends on the JavaBeans Activation Framework.

Let's add those dependencies in our POM:

<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.4.7</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
    <scope>runtime</scope>
</dependency>
We can find the latest versions of the Java Mail API and JavaBeans Activation Framework on Maven Central.

3. Configuring SMTPAppender
   Logback's SMTPAppender, by default, triggers an email when logging an ERROR event.

It holds all the logging events in a cyclic buffer with a default maximum capacity of 256 events. After the buffer gets full, it throws away any older log events.

Let's configure a SMTPAppender in our logback.xml:

<appender name="emailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>OUR-SMTP-HOST-ADDRESS</smtpHost>
    <!-- one or more recipients are possible -->
    <to>EMAIL-RECIPIENT-1</to>
    <to>EMAIL-RECIPIENT-2</to>
    <from>SENDER-EMAIL-ADDRESS</from>
    <subject>BAELDUNG: %logger{20} - %msg</subject>
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
    </layout>
</appender>
Also, we'll add this appender to our Logback configuration's root element:

<root level="INFO">
    <appender-ref ref="emailAppender"/>
</root>
As a result, for any application ERROR that gets logged, it'll send an email with all the buffered logging events formatted by PatternLayout.

We can further replace the PatternLayout with an HTMLLayout to format the log messages in an HTML table:


4. Custom Buffer Size
   We now know that by default, the outgoing email will contain the last 256 logging event messages. However, we can customize this behavior by including the cyclicBufferTracker configuration and specifying the desired bufferSize.

For triggering an email notification that'll only include the latest five logging events, we'll have:

<appender name="emailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>OUR-SMTP-HOST-ADDRESS</smtpHost>
    <to>EMAIL-RECIPIENT</to>
    <from>SENDER-EMAIL-ADDRESS</from>
    <subject>BAELDUNG: %logger{20} - %msg</subject>
    <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
    <cyclicBufferTracker class="ch.qos.logback.core.spi.CyclicBufferTracker"> 
        <bufferSize>5</bufferSize>
    </cyclicBufferTracker>
</appender>
5. SMTPAppender for Gmail
If we're using Gmail as our SMTP provider, we'll have to authenticate over SSL or STARTTLS.

To establish a connection over STARTTLS, the client first issues a STARTTLS command to the server. If the server supports this communication, the connection then switches over to SSL.

Let's now configure our appender for Gmail using STARTTLS:

<appender name="emailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>smtp.gmail.com</smtpHost>
    <smtpPort>587</smtpPort>
    <STARTTLS>true</STARTTLS>
    <asynchronousSending>false</asynchronousSending>
    <username>SENDER-EMAIL@gmail.com</username>
    <password>GMAIL-ACCT-PASSWORD</password>
    <to>EMAIL-RECIPIENT</to>
    <from>SENDER-EMAIL@gmail.com</from>
    <subject>BAELDUNG: %logger{20} - %msg</subject>
    <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
</appender>
