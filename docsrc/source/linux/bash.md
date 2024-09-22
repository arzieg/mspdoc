# Remarks on bash

https://quickref.me/bash


# Search String contain Substring
https://nickjanetakis.com/blog/check-if-a-string-contains-a-substring-in-bash

```
#!/usr/bin/env bash

set -o errexit
set -o pipefail
set -o nounset

ENV="${1:-prod}"
DEPLOYABLE_ENVS="${DEPLOYABLE_ENVS:-pen,prod}"

# This is the condition. We're using * to match anything on either side of the
# string Of course you can adjust this as needed for your case. It's 
# important that the wildcard is on the rigjt side of the condition.
if [[ "${DEPLOYABLE_ENVS}" == *"${ENV}"* ]]; then
	echo "Yes"
else
	echo "No"
	exit
fi
```
