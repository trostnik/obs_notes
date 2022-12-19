Under the hood, a HashMap uses a hash table data structure to store its key-value pairs. A hash table is an array of linked lists, where each linked list represents a bucket. The hash function is used to map a key to a specific bucket in the hash table.

When you add a key-value pair to a HashMap, the key is passed through the hash function, which calculates a unique integer based on the key. This integer is then used to determine the index of the bucket in the hash table where the value will be stored.

For example, if the hash function maps the key "apple" to the integer 3, and the hash table has 10 buckets, then the value will be stored in the third bucket.

If two keys are mapped to the same bucket, they are said to have a collision. In this case, the HashMap will use a linked list to store the values in the bucket, allowing multiple values to be stored in the same bucket.

When you try to retrieve a value from a HashMap using a key, the key is passed through the hash function again, and the resulting integer is used to determine the index of the bucket where the value is stored. The HashMap will then search the linked list in the bucket for the value associated with the key.

Overall, the use of a hash function and a hash table data structure allows HashMap to provide fast insertion, deletion, and lookup of values based on their keys. However, it is important to choose a good hash function to minimize the number of collisions and ensure good performance.