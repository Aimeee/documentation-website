---
layout: default
title: Go client
nav_order: 80
---

# Go client

The OpenSearch Go client lets you programmatically interact with data in your OpenSearch cluster as part of your Go application.


## Setup

If you're creating a new project:

```go
go mod init
```

To add the client to your project, import it like any other module:

```go
go get github.com/opensearch-project/opensearch-go
```

## Sample code

```go
package main

import (
	"context"
	"crypto/tls"
	"fmt"
	opensearch "github.com/opensearch-project/opensearch-go"
	opensearchapi "github.com/opensearch-project/opensearch-go/opensearchapi"
	"net/http"
	"strings"
)

const IndexName = "go-test-index1"

func main() {

	// Initialize the client with SSL/TLS enabled.
	client, err := opensearch.NewClient(opensearch.Config{
		Transport: &http.Transport{
			TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
		},
		Addresses: []string{"https://localhost:9200"},
		Username:  "admin", // For testing only. Don't store credentials in code.
		Password:  "admin",
	})
	if err != nil {
		fmt.Println("cannot initialize", err)
	}

	// Print OpenSearch version information on console.
	fmt.Println(client.Info())

	// Define a mapping.
	mapping := strings.NewReader(`{
	 'settings': {
	   'index': {
            'number_of_shards': 4
            }
          }
	 }`)

	// Create an index with non-default settings.
	res := opensearchapi.CreateRequest{
		Index: IndexName,
		Body:  mapping,
	}
	fmt.Println("creating index", res)

	// Add a document to the index.
	document := strings.NewReader(`{
		"title": "Moneyball",
		"director": "Bennett Miller",
		"year": "2011"
	}`)

	docId := "1"
	req := opensearchapi.IndexRequest{
		Index:      IndexName,
		DocumentID: docId,
		Body:       document,
	}
	insertResponse, err := req.Do(context.Background(), client)
	fmt.Println(insertResponse)

	// Search for the document.
	content := strings.NewReader(`{
  	   "size": 5,
  	   "query": {
    	   "multi_match": {
      	   "query": "miller",
      	   "fields": ["title^2", "director"]
    	   }
  	  }
	}`)

	search := opensearchapi.SearchRequest{
		Body: content,
	}

	searchResponse, err := search.Do(context.Background(), client)
	fmt.Println(searchResponse)

	// Delete the document.
	delete := opensearchapi.DeleteRequest{
		Index:      IndexName,
		DocumentID: docId,
	}

	deleteResponse, err := delete.Do(context.Background(), client)
	fmt.Println("deleting document")
	fmt.Println(deleteResponse)

	// Delete the index.
	deleteIndex := opensearchapi.IndicesDeleteRequest{
		Index: []string{IndexName},
	}

	deleteIndexResponse, err := deleteIndex.Do(context.Background(), client)
	fmt.Println("deleting index")
	fmt.Println(deleteIndexResponse)
}
```