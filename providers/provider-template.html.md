---
title: Your Service
---

Some description of what the service is good for.

## <a id='managing'></a>Creating and Managing Services Instances ##

[Managing services from the command line](/devguide/services/managing-services.html)

Is your service bindable? If not, you should say so.

## <a id='using'></a>Using Service Instances with your Application ##

Include this section only if your service is bindable. What is the format of the credentials stored in the VCAP_SERVICES environment variable?

See [Binding Applications to Service Instances](/devguide/services/application-binding.html) and [VCAP_SERVICES Environment Variable](/devguide/deploy-apps/environment-variable.html).

Format of credentials in `VCAP_SERVICES` environment variable.

~~~xml
{
  service-foo-n/a: [
    {
      name: "service-foo-75efc",
      label: "service-foo-n/a",
      plan: "example-plan",
      credentials: {
        uri: dbtype://username:password@hostname:port/name
        hostname: "foo.example.com"
        port: "1234"
        name: "asdfjasdf"
        username: "QvsXMbJ2rK",
        password: "HCDVOYluTv"
      }
    }
  ]
}
~~~

### Spring
Framework-specific integration techniques.

### Rails
Framework-specific integration techniques.

### Node.js
Framework-specific integration techniques.

## <a id='sample-app'></a>Sample Applications ##

Links and documentation for a sample app which a Cloud Foundry user could use to see the value of your service.

## <a id='dashboard'></a>Dashboard ##

What users can do with your dashboard.

## <a id='addl-docs'></a>Additional Documentation

URL to your docs site

## <a id='support'></a>Support ##

[Contacting Service Providers for Support](/marketplace/contacting-service-providers-for-support.html)

URL to your support page
