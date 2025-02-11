
```
Aaliyah is showing you how Intelligence Analysts work. She pulls up a piece of intelligence she thought was interesting. It shows that APTs are interested in acquiring hardware tokens used for accessing DIB networks. Those are generally controlled items, how could the APT get a hold of one of those?

DoD sometimes sends copies of procurement records for controlled items to the NSA for analysis. Aaliyah pulls up the records but realizes it’s in a file format she’s not familiar with. Can you help her look for anything suspicious?

If DIB companies are being actively targeted by an adversary the NSA needs to know about it so they can help mitigate the threat.

Help Aaliyah determine the outlying activity in the dataset given

Downloads:

- [DoD procurement records (shipping.db)](https://nsa-codebreaker.org/files/task1/shipping.db?1739284695)

---

Prompt:

- Provide the order id associated with the order most likely to be fraudulent.
```

Starting out my first thought was to view the DB file by just doing the following query `SELECT * From Zip`

When opening the shipping.db file I saw the first row is mimetype which indicates format of a file. In this row it states the file format is "application/vnd.oasis.OPENDOCUMENT.SPREADSHEET" or .ods

I will attempt to change the file format of shipping.db to shipping.ods and try and open the file.

Opening the file gives us what appears to be the actually shipping table full of shipping orders.
![[{E6CAA0A7-A2AA-4FD0-A6EC-215695A3C15F}.png]]

Applying a basic filter and filtering on company Aegis Defense solutions shows that the company appears to ship to the same location consistently. I am going to manually sift through and see if the other companies are the same. If so we can look for any outliers to be the fraudulent order.

After sifting through the companies we have found an outlier in Guardian Armaments orders seen below.
![[{DEB9303A-7E42-43BF-A9DD-A81923B58E74}.png]]

Entering the order number for Task 1 gives us the completion allowing us to proceed forward.

GUA0006453