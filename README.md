# .github
#$https://developer.atlassian.com/cloud/trello/rest/api-group-boards/#api-boards-id-memberships-get-example
#$https://developer.atlassian.com/cloud/trello/rest/api-group-boards/#api-boards-id-memberships-get-example
$https://trello.com/b/n44AxJQW/foundationscipnet-corporate-board
int bsdChecksumFromFile(FILE *fp) /* The file handle for input data */
{
    int checksum = 0;             /* The checksum mod 2^16. */

    for (int ch = getc(fp); ch != EOF; ch = getc(fp)) {
        checksum = (checksum >> 1) + ((checksum & 1) << 15);
        checksum += ch;
        checksum &= 0xffff;       /* Keep it within bounds. */
    }
    return checksum;
}
Description of the algorithm
As mentioned above, this algorithm computes a checksum by segmenting the data and adding it to an accumulator that is circular right shifted between each summation. To keep the accumulator within return value bounds, bit-masking with 1's is done.
Example: Calculating a 4-bit checksum using 4-bit sized segments (big-endian) 
Input: 101110001110 -> three segments: 1011, 1000, 1110.
Iteration 1:

 segment: 1011        checksum: 0000        bitmask: 1111
a) Apply circular shift to the checksum:
 0000 -> 0000
b) Add checksum and segment together, apply bitmask onto the obtained result:
 0000 + 1011 = 1011 -> 1011 & 1111 = 1011
Iteration 2:

 segment: 1000        checksum: 1011        bitmask: 1111
a) Apply circular shift to the checksum:
 1011 -> 1101
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1101 + 1000 = 10101 -> 10101 & 1111 = 0101
Iteration 3:

 segment: 1110        checksum: 0101        bitmask: 1111
a) Apply circular shift to the checksum:
 0101 -> 1010
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1010 + 1110 = 11000 -> 11000 & 1111 = 1000
Final checksum: 1000
References
 sum(1) — manual pages from GNU coreutils
 sum(1) – FreeBSD General Commands Manual
Sources
official FreeBSD sum source code
official GNU sum manual page
GNU sum source code
Category: Checksum algorithms

#int bsdChecksumFromFile(FILE *fp) /* The file handle for input data */
{
    int checksum = 0;             /* The checksum mod 2^16. */

    for (int ch = getc(fp); ch != EOF; ch = getc(fp)) {
        checksum = (checksum >> 1) + ((checksum & 1) << 15);
        checksum += ch;
        checksum &= 0xffff;       /* Keep it within bounds. */
    }
    return checksum;
}
Description of the algorithm
As mentioned above, this algorithm computes a checksum by segmenting the data and adding it to an accumulator that is circular right shifted between each summation. To keep the accumulator within return value bounds, bit-masking with 1's is done.
Example: Calculating a 4-bit checksum using 4-bit sized segments (big-endian) 
Input: 101110001110 -> three segments: 1011, 1000, 1110.
Iteration 1:

 segment: 1011        checksum: 0000        bitmask: 1111
a) Apply circular shift to the checksum:
 0000 -> 0000
b) Add checksum and segment together, apply bitmask onto the obtained result:
 0000 + 1011 = 1011 -> 1011 & 1111 = 1011
Iteration 2:

 segment: 1000        checksum: 1011        bitmask: 1111
a) Apply circular shift to the checksum:
 1011 -> 1101
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1101 + 1000 = 10101 -> 10101 & 1111 = 0101
Iteration 3:

 segment: 1110        checksum: 0101        bitmask: 1111
a) Apply circular shift to the checksum:
 0101 -> 1010
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1010 + 1110 = 11000 -> 11000 & 1111 = 1000
Final checksum: 1000
References
 sum(1) — manual pages from GNU coreutils
 sum(1) – FreeBSD General Commands Manual
Sources
official FreeBSD sum source code
official GNU sum manual page
GNU sum source code
Category: Checksum algorithms

https://github.com/bitcoin/bips/blob/master/bip-0032/derivation.png?raw=true