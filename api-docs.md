# Pavebyte Documentation

Pavebyte is a prepaid cloud storage service. The pricing is 5 EUR/TB/mo, charged per byte and in increments of 864 minutes.

## Creating an account

```bash
curl https://pavebyte.com/api/register -F "a=1"
```

A field with the name `a` is required.

The response is the user ID in plain text.

## Querying balance

```bash
curl "https://pavebyte.com/api/user_info?uid=USER_ID"
```

Example response:

```json
{"balance":100000000000000}
```

In this case, the balance is 10 euros:

```lua
local balance = 100000000000000
print(string.format("%d.%013d euros", balance // 10000000000000, balance % 10000000000000)) --> 10.00000000000000 euros
```

## Adding to your balance

You can top up your balance by redeeming a giftcard code. You can buy giftcard codes with cryptocurrency [here](https://calamity-inc.mysellix.io/product/pavebyte-5-euro-giftcard).

```bash
curl https://pavebyte.com/api/giftcard_claim -F "uid=USER_ID" -F "giftcard=GIFTCARD_CODE"
```

Returns the new account balance in plain text.

## Creating giftcards

```bash
curl https://pavebyte.com/api/giftcard_create -F "uid=USER_ID" -F "amount=50000000000000" # 5 euros
```

An optional "count" field can be provided to produce multiple giftcards.

On success:
- Reduces the account's balance by `amount * count`.
- Returns a JSON array containing the giftcard codes.

## Creating folders

```bash
# Creating a root folder
curl https://pavebyte.com/api/create_root_folder -F "uid=USER_ID"

# Creating a sub-folder
curl https://pavebyte.com/api/create_sub_folder -F "uid=USER_ID" -F "folder=FOLDER_ID" -F "name=My Sub Folder"
```

The response is the folder ID in plain text.

## Creating files

```bash
# Using one or more local files
curl https://s1.pavebyte.com/api/upload -F "uid=USER_ID" -F "folder=FOLDER_ID" -F "files[]=@hello.txt"

# Using a file that's already on the web
curl https://s1.pavebyte.com/api/reverse_upload -F "uid=USER_ID" -F "folder=FOLDER_ID" -F "name=image.jpg" -F "url=https://i.redd.it/0c02ptapc0k21.jpg"
```

The file(s) will have an initial lifetime of 864 minutes. You can optionally provide a "timescale" field to serve as a multiplier for this. For example, `-F "timescale=50"` will set an initial lifetime of 30 days.

On success:
- Reduces the account's balance by `total_content_bytes * timescale`.
- A JSON array of file IDs is returned.

Important note about sharing the response data: The file ID alone can be used to delete the respective file.

## Disabling public directory listing

Root folders are "unlisted", but if someone knows its ID, they can access it and get a listing of its sub-folders.

If you don't want a folder to provide a listing, upload an index.html into it. This file can be empty, which means you can create it for free.

## Listing files and folders in a folder

```bash
curl https://pavebyte.com/api/ls -F "uid=USER_ID" -F "folder=FOLDER_ID"
```

Example response:

```json
{"folders":[{"id":"gy502n2d","name":"My Sub Folder"}],"files":[{"id":"l02fusg6kidy","name":"image.jpg","expiry":1730756825,"size":162321,"sha256":"e68ac0dc99d64f88e26907e51268ffdcbcf8266a42a3b23da3f33272c0337683"}]}
```

Important note about sharing the response data: The file ID alone can be used to delete the respective file.

## Extending a file's lifetime

```bash
curl https://pavebyte.com/api/renew -F "uid=USER_ID" -F "file=FILE_ID" -F "timescale=50"
```

Increases the file's lifetime in 864-minute intervals. The "timescale" field is an optional multiplier. `-F "timescale=50"` equals 30 days.

On success:
- Reduces the account's balance by `file_size * timescale`.
- The new expiry timestamp is returned in plain text.

## Deleting a file

```bash
curl https://pavebyte.com/api/delete -F "file=FILE_ID"
```

Note: Unused time is not refunded.
