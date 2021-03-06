# concourse-json-api-resource

A concourse resource for querying JSON based API over HTTP.

This resource implements `check`, `in` and `out` functions.

The 'check' step queries the API and will search for a `dpath` style dictionary path within the JSON response obtained from the API. The `json_path` parameter is used to select which value from the JSON response will be used as the resource `version` which is sent back to Concourse.

This resource only implements POST requests to an API. The query to send to the API is contained within the `post_data` and is expected to generate a JSON response from the API. The `json_path` is used as a query path within the JSON response to extract a version number to report back to Concourse. We didn't require GET functionality for the API we're talking to when we created this resource, so we never implemented it. (Pull requests are welcomed if you want to add the functionality)

This resource accepts the following inputs:

* `source.url` : The URL to send queries to
* `source.verify_ssl` : True or False (to skip SSL verification checks)
* `source.auth_token` : Bearer token to be used when communicating with `source.url`
* `source.post_data` : The data to be sent to the `source.url` as part of a POST request.
* `source.content_type` : Set the content type to use in the headers (e.g. `application/json`)
* `source.json_path` : The json element key holding the data to process
* `source.version_key` : The json element key holding the version to be used as reference is Concourse
* `source.file_name` : The `file_name` to create and save the JSON api response data into. (Just the file name, no path allowed)

## Pipeline example

```yaml
resource_types:
- name: json_api_resource
  source:
    ca_certs:
    repository: dsmule/concourse-json-api-resource
    tag: 0.9
  type: docker-image

resources:
- name: api-resource-example
  type: json_api_resource
  source:
    url: https://some.api.server.com/somepath/rest/api/v1/gen/qry/data/
    verify_ssl: False
    auth_token: AbcD1234FgHKJl
    post_data: '{"code": "EXECUTE_API_OPERATION"}'
    content_type: application/data
    json_path: RESPONSE_DATA_TARGETS
    version_key: TARGET_ID
    file_name: api_data.json
    debug: true



```

## development

### pre-reqs

`pip install -r requirements.txt` to get all of the required python libraries installed.

### testing

Unit tests exist for all functions.
Run `pytest -v` to confirm that all tests pass before you make any changes, and run it again after changes to confirm that you've not broken anything.

To test things out on the command line (outside of a Concourse pipeline) when doing development work, you will need to supply a suitable `payload` on stdin.

e.g `echo '{"source": {"url": "https://some.url/some/path/to/use", "verify_ssl": "False", "auth_token":"<bearer_token>", "post_data":"{\"code\": \"CALL_TO_API_FUNCTION\"}", "content_type":"application/json", "json_path":"/path/to/key/in/json/dict"}}' | ./check.py`
