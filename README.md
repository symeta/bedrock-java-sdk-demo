# bedrock-java-sdk-demo

## 1. pre-requisites
1.1 configure pom.xml @dev host
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <aws.java.sdk.version>2.20.160</aws.java.sdk.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bedrock</artifactId>
            <version>2.20.160</version>
        </dependency>

        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-bedrock</artifactId>
            <version>1.12.562</version>
        </dependency>

        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bedrockruntime</artifactId>
        <version>2.20.160</version>
        
</dependency>
    </dependencies>
    
</project>
```
1.2 configure aws account credential @dev host
- install aws cli @dev host
```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
- configure credentials
```sh
aws configure
#input IAM user's AK following the instruction guidance
#input IAM user's SK following the instruction guidance
#input region id that has enabled Bedrock, e.g. us-west-2
```
1.3 configure IAM user access Bedrock permission
- add an inline policy for the IAM user to authenticate the user to use Bedrock
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": "bedrock:*",
			"Resource": "*"
		}
	]
}
```

## 2. java project code
2.1 main class
```java
package com.example;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.concurrent.ExecutionException;

public class app {
    private static final Logger logger = LoggerFactory.getLogger(app.class);

    public static void main(String... args) {
        logger.info("Application starts");

        final bedrockTest client = new bedrockTest();
        try {
            client.runStream(); 
          } catch (ExecutionException e) {
            logger.info("runStream ExecutionException");
          } catch (InterruptedException e) {
            logger.info("runStream InterruptedException");
          }
        

        logger.info("Application ends");
    }
    
}
```
2.2 bedrockTest class
```java
package com.example;

import software.amazon.awssdk.core.SdkBytes;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.bedrockruntime.BedrockRuntimeAsyncClient;
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
import software.amazon.awssdk.services.bedrockruntime.model.*;
import java.net.URI;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.ExecutionException;

public class bedrockTest {

    private final BedrockRuntimeAsyncClient bedrockClient = BedrockRuntimeAsyncClient.builder()
                .endpointOverride(URI.create("https://bedrock-runtime.us-west-2.amazonaws.com"))
                .region(Region.US_WEST_2)
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();    

    //private final BedrockRuntimeAsyncClient bedrockClient = BedrockRuntimeAsyncClient.create();
    public void runStream() throws ExecutionException, InterruptedException {
        final String bodyString = "{\"prompt\": \"\\n\\nHuman:write an essay for living on mars in 1000 words\\n\\nAssistant:\", \"max_tokens_to_sample\": 2000}";
        SdkBytes bodyBytes = SdkBytes.fromUtf8String(bodyString);
        try {
            InvokeModelWithResponseStreamRequest request = InvokeModelWithResponseStreamRequest.builder()
                                                    .accept("application/json") 
                                                    .body(bodyBytes) 
                                                    .contentType("application/json") 
                                                    .modelId("anthropic.claude-v2")
                                                    .build(); 
            InvokeModelWithResponseStreamResponseHandler handler = InvokeModelWithResponseStreamResponseHandler.builder()
                                                    .onEventStream(publisher ->publisher.subscribe(bedrockTest::processEvent))
                                                    .build();
            bedrockClient.invokeModelWithResponseStream(request, handler).get();
            
        } 
        catch (ExecutionException e) {
                System.out.println("InvokeModelWithResponseStreamRequest ExecutionException");
            } 
        catch (InterruptedException e) {
                System.out.println("InvokeModelWithResponseStreamRequest InterruptedException");
            }
        }

    private static void processEvent(ResponseStream event) {
        if (event instanceof PayloadPart) {
            final PayloadPart payloadPart = (PayloadPart) event;
            final String message =
                         StandardCharsets.UTF_8.decode(payloadPart.bytes().asByteBuffer()).toString();
                        System.out.println(message);
        }
    }

}
```

## 3. aws bedrock java sdk package definition

[bedrock java sdk package difinition](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/bedrock/package-summary.html)
