class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        return solve(s,wordDict);
    }

    private boolean solve(String s, List<String> wordDict){
        if (s.length() == 0) return true;
        
        for(int i=1;i<=s.length();i++){
            if(wordDict.contains(s.substring(0,i))){
                if(solve(s.substring(i),wordDict)) return true;
            }
        }
        
        return false;

       
    }
}
