Problem: Print all binary strings of length n
Given n = 3, print all strings made of 0 and 1 of that length.
Expected output (order doesn't matter):
000, 001, 010, 011, 100, 101, 110, 111



void function(int n, int k, List<String> result, StringBuilder sb) {
    if (k == n) {
        result.add(sb.toString());    // base case — all slots filled
        return;
    }

    sb.append('0');                   // CHOOSE 0
    function(n, k+1, result, sb);     // EXPLORE
    sb.deleteCharAt(sb.length()-1);   // UNCHOOSE

    sb.append('1');                   // CHOOSE 1
    function(n, k+1, result, sb);     // EXPLORE
    sb.deleteCharAt(sb.length()-1);   // UNCHOOSE
}
