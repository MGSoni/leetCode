class Solution {
    public List<List<Integer>> pacificAtlantic(int[][] heights) {

        Queue<int[]> pacificQueue = new LinkedList<>();
        Queue<int[]> atlanticQueue = new LinkedList<>();

        boolean[][] pacificVisited = new boolean[heights.length][heights[0].length];
        boolean[][] atlanticVisited = new boolean[heights.length][heights[0].length];

        for(int j=0;j<heights[0].length;j++){
            int[] p = new int[]{0,j};
            int[] a = new int[]{heights.length-1,j};
            if(!pacificVisited[0][j]){
                pacificQueue.add(p);
                pacificVisited[0][j] = true;
            }
            if(!atlanticVisited[heights.length-1][j]){
                atlanticQueue.add(a);
                atlanticVisited[heights.length-1][j] = true;
            }  
        }

        for(int i=0;i<heights.length;i++){
            int[] p = new int[]{i,0};
            int[] a = new int[]{i,heights[0].length-1};
            if(!pacificVisited[i][0]){
                pacificQueue.add(p);
                pacificVisited[i][0] = true;
            }
            if(!atlanticVisited[i][heights[0].length-1]){
                atlanticQueue.add(a);
                atlanticVisited[i][heights[0].length-1] = true;
            }
        }
        
        bfs(heights, pacificVisited, pacificQueue);
        bfs(heights, atlanticVisited, atlanticQueue);
        List<List<Integer>> result = new ArrayList<>();
        for(int i=0;i<heights.length;i++){
            for(int j=0;j<heights[0].length;j++){
                if(pacificVisited[i][j] && atlanticVisited[i][j]){
                    result.add(List.of(i,j));
                }
            }
        }
        return result;
    }

    private void bfs(int[][] heights, boolean[][]visited, Queue<int[]> queue){
        int[] rowArr = new int[]{1,0,-1,0};
        int[] colArr = new int[]{0,1,0,-1};

        while(!queue.isEmpty()){
            int[] arr = queue.remove();
            int row = arr[0];
            int col = arr[1];
            for(int i=0;i<4;i++){
                int newRow = row+rowArr[i];
                int newCol = col+colArr[i];
                if(newRow >=0 && newRow < heights.length && newCol>=0 && newCol< heights[0].length && heights[newRow][newCol] >= heights[row][col]){
                    if(!visited[newRow][newCol]){
                        queue.add(new int[]{newRow, newCol});
                        visited[newRow][newCol] = true;
                    }
                }
            }
        }
    }
}
