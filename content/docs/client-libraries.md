+++
title = "Client Libraries"
+++

# Client Libraries


## Go

You can find the Go client library for Pilosa at our [Go Pilosa Repository](https://github.com/pilosa/go-pilosa). Check out its [README](https://github.com/pilosa/go-pilosa/README.md) for more information and installation instructions.

We are going to use the index you have created in the [Getting Started](getting_started) section. Before carrying on, make sure that example index is created and Pilosa server is running on the default address: `http://localhost:10101`.

Note that, there are no error handling in the example below for brevity purpose.

```go
package startrace

import (
    "fmt"

    pilosa "github.com/pilosa/go-client-pilosa"
)

func main() {
    // Let's create Index and Frame objects, which will contain the settings
    // for the corresponding indexes and frames.
    repositoryOptions, _ := pilosa.ColumnLabelIndexOption("repo_id")
    repository, _ := pilosa.NewIndex("repository", repositoryOptions)

    stargazerOptions, _ := pilosa.RowLabelFrameOption("stargazer_id")
    stargazer, _ := repository.Frame("stargazer", stargazerOptions)

    languageOptions, _ := pilosa.RowLabelFrameOption("language_id")
    language, _ := repository.Frame("language", languageOptions)

    // We will just use the default client which assumes the server is at http://localhost:10101
    client := pilosa.DefaultClient()

    var response *pilosa.QueryResponse
    var result *pilosa.QueryResult

    // Which repositories did user 8 star:
    response, _ = client.Query(stargazer.Bitmap(8), nil)
    result = response.Result()
    if result != nil {
        fmt.Println("User 8 starred: ", result.Bitmap.Bits)
    }

    // What are the top 5 languages in the sample data:
    response, _ = client.Query(language.TopN(5), nil)
    if result != nil {
        fmt.Println("Top 5 languages: ", result.Bitmap.Bits)
    }

    // Which repositories were starred by user 8 and 18:
    response, _ = client.Query(
        repository.Intersect(
            stargazer.Bitmap(8),
            stargazer.Bitmap(18)),
        nil)
    result = response.Result()
    if result != nil {
        fmt.Println("Repositories starred by both user 8 and 18: ", result.Bitmap.Bits)
    }

    // Which repositories were starred by user 8 and 18 and also were written in language 1
    response, _ = client.Query(
        repository.Intersect(
            stargazer.Bitmap(8),
            stargazer.Bitmap(18),
            language.Bitmap(1)),
        nil)
    result = response.Result()
    if result != nil {
        fmt.Println("Repositories starred by both user 8 and 18 and are in language 1: ", result.Bitmap.Bits)
    }

    // Set user 99999 as a stargazer for repository 77777:
    response, _ = client.Query(stargazer.SetBit(99999, 77777), nil)
    if !response.Success {
        fmt.Println("Error setting bit: ", response.ErrorMessage)
    }
}
```

## Python

You can find the Python client library for Pilosa at our [Python Pilosa Repository](https://github.com/pilosa/python-pilosa). Check out its [README](https://github.com/pilosa/python-pilosa/README.md) for more information and installation instructions.

We are going to use the index you have created in the [Getting Started](getting_started) section. Before carrying on, make sure that example index is created and Pilosa server is running on the default address: `http://localhost:10101`.

Note that, there are no error handling in the example below for brevity purpose.

```python
from pilosa import Index, Frame, Client

# Let's create Index and Frame objects, which will contain the settings
# for the corresponding indexes and frames.
repository = Index("repository", column_label="repo_id")
stargazer = repository.Frame("stargazer", row_label="stargazer_id")
language = repository.Frame("language", row_label="language_id")

# We will just use the default client which assumes the server is at http://localhost:10101
client = Client()

# Which repositories did user 8 star:
response = client.query(stargazer.bitmap(8))
if response.result:
    print("User 8 starred: ", result.bitmap.bits)

# What are the top 5 languages in the sample data:
response = client.query(language.topn(5))
if response.result:
    print("Top 5 languages: ", result.bitmap.bits)

# Which repositories were starred by user 8 and 18:
response = client.query(
        repository.intersect(
            stargazer.bitmap(8),
            stargazer.bitmap(18)))
if response.result:
    print("Repositories starred by both user 8 and 18: ", result.bitmap.bits)

# Which repositories were starred by user 8 and 18 and also were written in language 1
response = client.query(
        repository.intersect(
            stargazer.bitmap(8),
            stargazer.bitmap(18),
            language.bitmap(1)))
if response.result:
    print("Repositories starred by both user 8 and 18 and are in language 1: ", result.bitmap.bits)

# Set user 99999 as a stargazer for repository 77777
response = client.query(stargazer.setbit(99999, 77777))
if not response.success:
    print("Error setting bit: ", response.error_message)

```

## Java

You can find the Java client library for Pilosa at our [Java Pilosa Repository](https://github.com/pilosa/java-pilosa). Check out its [README](https://github.com/pilosa/java-pilosa/README.md) for more information and installation instructions.

We are going to use the index you have created in the [Getting Started](getting_started) section. Before carrying on, make sure that example index is created and Pilosa server is running on the default address: `http://localhost:10101`.

Note that, there are no error handling in the example below for brevity purpose.

```java
import com.pilosa.client.*;
import com.pilosa.client.orm.*;

public class StarTrace {
    public static void main(String[] args) {
        // Let's create Index and Frame objects, which will contain the settings
        // for the corresponding indexes and frames.
        IndexOptions repositoryOptions = IndexOptions.builder()
            .setColumnLabel("repo_id")
            .build();
        Index repository = Index.withName("repository", repositoryOptions);

        FrameOptions stargazerOptions = FrameOptions.builder()
            .setRowLabel("stargazer_id")
            .build();
        Frame stargazer = repository.frame("stargazer", stargazerOptions);

        FrameOptions languageOptions = FrameOptions.builder()
            .setRowLabel("language_id")
            .build();        
        Frame language = repository.frame("language", languageOptions);

        // We will just use the default client which assumes the server is at http://localhost:10101
        PilosaClient client = PilosaClient.defaultClient();

        QueryResponse response;
        QueryResult result;

        // Which repositories did user 8 star:
        response = client.query(stargazer.bitmap(8));
        result = response.getResult();
        if (result != null) {
            System.out.println("User 8 starred: " + result.getBitmap().getBits());
        }

        // What are the top 5 languages in the sample data:
        response = client.query(language.topN(5));
        result = response.getResult();
        if (result != null) {
            System.out.println("Top 5 languages: " + result.getBitmap().getBits());
        }

        // Which repositories were starred by user 8 and 18:
        response = client.query(
            repository.intersect(
                stargazer.bitmap(8),
                stargazer.bitmap(18)));
        result = response.getResult();
        if (result != null) {
            System.out.println("Repositories starred by both user 8 and 18: "
                + result.getBitmap().getBits());
        }

        // Which repositories were starred by user 8 and 18 and also were written in language 1
        response = client.query(
            repository.intersect(
                stargazer.bitmap(8),
                stargazer.bitmap(18),
                language.bitmap(1)));
        result = response.getResult();
        if (result != null) {
            System.out.println("Repositories starred by both user 8 and 18 and are in language 1: "
                + result.getBitmap().getBits());
        }

        // Set user 99999 as a stargazer for repository 77777:
        response = client.query(stargazer.setBit(99999, 77777))
        if (!response.isSuccess()) {
            System.out.println("Error setting bit: " + response.getErrorMessage());
        }
    }
}
```