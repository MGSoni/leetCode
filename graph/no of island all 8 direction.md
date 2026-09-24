public class Solution {
    public int solve(int[][] grid) {
         Queue<int[]> visited = new LinkedList<int[]>();
         int result = 0;

        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j] == 1){
                    result++;
                    visited.add(new int[]{i,j});
                    grid[i][j] = 0;
                    while(!visited.isEmpty()){
                        int[] arr = visited.remove();
                        int row = arr[0];
                        int col = arr[1];
                        if(row+1 < grid.length && grid[row+1][col] == 1){
                             visited.add(new int[]{row+1,col});
                                grid[row+1][col] = 0;
                        }
                        if(row-1>=0 && grid[row-1][col] == 1){
                             visited.add(new int[]{row-1,col});
                                grid[row-1][col] = 0;
                        }
                        if(col+1<grid[0].length && grid[row][col+1] == 1){
                             visited.add(new int[]{row,col+1});
                                grid[row][col+1] = 0;
                        }
                        if(col-1>=0 && grid[row][col-1] == 1){
                             visited.add(new int[]{row,col-1});
                                grid[row][col-1] = 0;
                        }
                        if(row+1 < grid.length && col+1<grid[0].length && grid[row+1][col+1] == 1){
                             visited.add(new int[]{row+1,col+1});
                                grid[row+1][col+1] = 0;
                        }
                        if(row-1>=0 && col+1<grid[0].length && grid[row-1][col+1] == 1){
                             visited.add(new int[]{row-1,col+1});
                                grid[row-1][col+1] = 0;
                        }
                        if(col-1>=0 && row-1>=0 && grid[row-1][col-1] == 1){
                             visited.add(new int[]{row-1,col-1});
                                grid[row-1][col-1] = 0;
                        }
                        if(col-1>=0 && row+1 < grid.length && grid[row+1][col-1] == 1){
                             visited.add(new int[]{row+1,col-1});
                                grid[row+1][col-1] = 0;
                        }
                    }
                    
                }
            }
        }
        return result;
    }
}
