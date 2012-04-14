# fdoc
## Farnsdocs: Documentation format and verification

High-quality documentation is extremely useful, but maintaining it is often a pain. We aim to create a tool to facilitate easy creation and maintenance of API documentation.

Fdoc is named for everybody's favorite, good news-bearing, crotchety old man, Professor Farnsworth.

![Professor Farnsworth](https://github.com/square/fdoc/raw/master/docs/farnsworth.png)

### Example

`.fdoc` files are YAML files based on JSON schema to describe API resources and their endpoints.

    resourceName: members
    basePath: https://api.example.com/members
    description: Provides access to the members of Example company.
    controller: Api::MembersController
    actions:
        -   name: list
            verb: GET
            description: The list of members.
            requestParameters:
                properties:
                    limit:
                        type: integer
                        required: no
                        default: 50
                        description: Limits the number of results returned, used for paging.
            responseParameters:
                properties:
                    members:
                        type: array
                        items:
                            title: member
                            description: Representation of a member
                            type: object
                            properties:
                                name:
                                    description: Member's name
                                    type: string
                                    required: yes
                                    example: Captain Smellypants
            responseCodes:
                -   status: 200 OK
                    successful: yes
                    description: A list of current members
                -   status: 400 Bad Request
                    successful: no
                    description: Indicates malformed parameters
    

### Goals

 - As a client engineer, I want to be able to document an API and keep it up to date
 - The server engineers want to be able to test their implementations
 - The documentation should be as close to the code as possible, it should live in the same repository (like specs)
   - Branches, reviews, and merges are the appropriate way to update the docs
   - Experimental drafts should just live on branches and never get merged into master
 - Specification alone is not enough, there needs to be room for discussion

### Usage

Fdoc creates an object graph based on `.fdoc` files in a directory when it is loaded (`Fdoc::load`), representing the source of truth. The format is specified as a [JSON schema](http://json-schema.org/), in `fdoc-schema.yaml`.

The `Action` class is the verifier of this source of truth, by wrapping around an individual method on an endpoint. Use `#consume_request` and `#consume_response`, and it will return `true` if input data matches expectations, or raise some sort of error if there is an incongruency. The intention is that this checklist is run in a spec environment in a project, to intercept usage of an endpoint.

Alternatively, the `Action` class can help create documentation. Its `#scaffold_request` and `#scaffold_response` take the exact same arguments as those their verifying equivalents, but will absorb all incoming parameters and update the object graph. To write this object graph to disk, use `Fdoc::Resource::write_to_directory`

### Hierarchy

In fdoc, the top level object in a graph is a Resource -- there is typically a one-to-one relation between Resources and individual `.fdoc` files. Resources typically represent groups of API endpoints (such as `/members`).

Resources can have multiple Actions, such as `GET list` or `POST add`. Methods represent individual endpoints and are essentially the atomic unit of testing and are often identified by their `(verb, name)` tuple.

Actions have:

- Request Parameters: a schema describing the expected format of a request.

- Response Parameters: a schema describing the expected format of a response.

- Response Codes: Response code-success pairings that enumerate possible responses.

### Feedback

Since fdoc is built on top of JSON schemas, all the hard work of verifiying that inputs conform their respective schemas is done by a [JSON schema gem](https://github.com/hoxworth/json-schema).

To make feedback more valuable, the request and response consumption methods will modify schemas to set `additionalProperties` to `false` unless specified. This gives the desired behavior of throwing an error when a new property is detected in the schema to verify, indicating the documentation needs updating.

