```
```
#import <Foundation/Foundation.h>

int main (int argc, const char * argv[])
{
   NSString *s1 = @"我的白日依山尽";
   NSString *s2 = @"你的白日依山经";
   int len1 = [s1 length];
   int len2 = [s2 length];
   int i = 0;
   int j = 0;
   int matrix[len1+1][len2+1]; 
   for(i=0;i<len1+1;i++){
    matrix[i][0] = i;
   }
   for(j=0;j<len2+1;j++){
    matrix[0][j] = j;
   }
   for(i=1;i<len1+1;i++){
    for(j=1;j<len2+1;j++){
        int a = matrix[i-1][j]+1;
        int b = matrix[i][j-1]+1;
        int c = matrix[i-1][j-1]+1;
        if([s1 characterAtIndex:(i-1)] == [s2 characterAtIndex:(j-1)]){
            c-=1;
        }
        // find min value
        int min = a;
        if(min>b){
            min = b;
        }
        if(min>c){
            min = c;
        }
        matrix[i][j]=min;
    }
   }
   // print matrix
   for(i=0;i<len1+1;i++){
    for(j=0;j<len2+1;j++){
        printf("%d\t",matrix[i][j]);
    }
    printf("\n");
   }
   NSLog(@"edit distance is %d",matrix[len1][len2]);
   return 0;
}

```
```
