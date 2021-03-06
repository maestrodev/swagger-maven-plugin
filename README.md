*You can:*
  - *Go to check out [this simple example of using this plugin](https://github.com/kongchen/swagger-maven-example) to see how it works*
  - *Or start reading following balabala:*

# Why this plugin

First of all, Swagger is awesome!

However, a limitation in using Swagger is that you cannot get your API document unless you:

1. write documents with Swagger style
* configure servlet
* build your project
* deploy your project on server
* start the server
* call to the servlet to let Swagger listing your APIs.

Another thing is you can get beautiful document by Swagger-ui, but you cannot get the format you need easily. e.g. A simple single markdown page.

This plugin will resolve these problems.
It supports customized output template (mustache) and will let you generate the document in build phase, in another word, you'll get the API document by following steps:

1. write documents with Swagger style
* configure this plugin
* build (mvn compile)

You'll always get your up to date document after you launch _mvn compile_. You can easily put the generated document anywhere. (put in service war package using maven-assembly-plugin.)

Check out [this sample page](https://github.com/kongchen/swagger-maven-plugin/wiki/Sample.markdown) which is a markdown file generated by this plugin (with a emmbed mustache template).

# Tutorials

Only **3 steps**, simple enough?

## 1. Write Swagger-style document
Follow [Swagger](https://github.com/wordnik/swagger-core/wiki/) tutorials, write your own API documents nearby your source code:

```java
@Path("/pet.json")
@Api(value = "/pet", description = "Operations about pets")
@Produces({"application/json"})
public class PetResource {
  @GET
  @Path("/{petId}")
  @ApiOperation(value = "Find pet by ID", notes = "Add extra notes here", responseClass = "com.wordnik.swagger.sample.model.Pet")
    @ApiErrors(value = { @ApiError(code = 400, reason = "Invalid ID supplied"),
    @ApiError(code = 404, reason = "Pet not found") })
  public Response getPetById (
    @ApiParam(value = "ID of pet that needs to be fetched", allowableValues = "range[1,5]", required = true) @PathParam("petId") String petId)
  throws NotFoundException {
    // your resource logic 
  }
```


## 2. Configure pom.xml
### 2.1 Add plugin repository
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    …
    <pluginRepositories>
        <pluginRepository>
            <id>swagger-maven-plugin-mvn-repo</id>
            <url>https://github.com/kongchen/swagger-maven-plugin/raw/mvn-repo/</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
    …
</project>
```

### 2.2 Configure plugin

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    …
    <build>
        <plugins>
            …
          <plugin>
            <groupId>com.github.kongchen</groupId>
            <artifactId>swagger-maven-plugin</artifactId>
            <version>1.0-SNAPSHOT</version>
            <configuration>
                <apiSources>
                    <!--can speficy multi sources here to get multi format outputs-->
                    <apiSource>
                        <apiClasses>
                            <apiClass>com.foo.bar.ApiResource1Version1</apiClass>
                            <apiClass>com.foo.bar.ApiResource2Version1</apiClass>
                        </apiClasses>
                        <apiPackage>com.foo.bar.api</apiPackage>
                        <apiVersion>v1</apiVersion>
                        <basePath>http://host:port/foo/bar</basePath>
                        <outputTemplate>markdown.mustache</outputTemplate>
                        <outputPath>doc.md</outputPath>
                    </apiSource>
                </apiSources>
            </configuration>
            <executions>
                <execution>
                    <phase>compile</phase>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
            …
        </plugins>
    </build>
</project>
```


> - A Java Class which contains Swagger's annotation @Api will be considered as a **apiClass**.
- You can specify several **apiClass** or an **apiPackage** which contains several **apiClass** to an **apiSource**.
- One **apiSource** will be considered as a set of APIs for one **apiVersion** in **basePath**.
- The **basePath** is only used to document, no used for call.
- You can specify several **apiSource** to the plugin. For example:
 - You can generate documents for your mutil-version supported service.
 - Or you can generate several formats of output for one version of your API.
- **outputTemplate** is the path of the mustache template file.
- **outputPath** is the path of the output file.

## 3. Build 
    mvn compile
You'll get your document ouput file in the path you specifid in **outputPath**.

# About the template file
Don't worry about the template file, the plugin has embed 3 templates in:

1. [wiki.mustache](https://github.com/kongchen/swagger-maven-plugin/blob/master/src/main/resources/wiki.mustache) for wiki markup output.
2. [html.mustache](https://github.com/kongchen/swagger-maven-plugin/blob/master/src/main/resources/html.mustache) for html output.
3. [markdown.mustache](https://github.com/kongchen/swagger-maven-plugin/blob/master/src/main/resources/markdown.mustache) for markdown output.

You can directly use them in **outputTemplate**, no extra path is needed, just the filename will be the path. e.g.:
```xml
<outputTemplate>markdown.mustache</outputTemplate>
```

If you dissatisfied with the included mustache template, you can write your own, just follow the following json-schema of the hash your mustache template file will consume. It looks big but actually simple:
```json
{
    "type": "object",
    "properties": {
        "basePath": {
            "type": "string"
        },
        "apiVersion": {
            "type": "string"
        },
        "apiDocuments": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "index": {
                        "type": "integer"
                    },
                    "resourcePath": {
                        "type": "string"
                    },
                    "description": {
                        "type": "string"
                    },
                    "apis": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "apiIndex": {
                                    "type": "integer"
                                },
                                "path": {
                                    "type": "string"
                                },
                                "url": {
                                    "type": "string"
                                },
                                "operations": {
                                    "type": "array",
                                    "items": {
                                        "type": "object",
                                        "properties": {
                                            "opIndex": {
                                                "type": "integer"
                                            },
                                            "httpMethod": {
                                                "type": "string"
                                            },
                                            "summary": {
                                                "type": "string"
                                            },
                                            "notes": {
                                                "type": "string"
                                            },
                                            "responseClass": {
                                                "type": "string"
                                            },
                                            "nickname": {
                                                "type": "string"
                                            },
                                            "parameters": {
                                                "type": "array",
                                                "items": {
                                                    "type": "object",
                                                    "properties": {
                                                        "paramType": {
                                                            "type": "string"
                                                        },
                                                        "paras": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "object",
                                                                "properties": {
                                                                    "name": {
                                                                        "type": "string"
                                                                    },
                                                                    "required": {
                                                                        "type": "boolean",
                                                                        "required": true
                                                                    },
                                                                    "description": {
                                                                        "type": "string"
                                                                    },
                                                                    "type": {
                                                                        "type": "string"
                                                                    },
                                                                    "linkType": {
                                                                        "type": "string"
                                                                    }
                                                                }
                                                            }
                                                        }
                                                    }
                                                }
                                            },
                                            "errorResponses": {
                                                "type": "array",
                                                "items": {
                                                    "type": "object",
                                                    "properties": {
                                                        "code": {
                                                            "type": "integer"
                                                        },
                                                        "reason": {
                                                            "type": "string"
                                                        }
                                                    }
                                                }
                                            },
                                            "responseClassLinkType": {
                                                "type": "string"
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        },
        "dataTypes": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {
                        "type": "string"
                    },
                    "items": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "name": {
                                    "type": "string"
                                },
                                "type": {
                                    "type": "string"
                                },
                                "linkType": {
                                    "type": "string"
                                },
                                "required": {
                                    "type": "boolean",
                                    "required": true
                                },
                                "access": {
                                    "type": "string"
                                },
                                "description": {
                                    "type": "string"
                                },
                                "notes": {
                                    "type": "string"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
