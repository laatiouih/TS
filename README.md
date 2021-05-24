# TS
#include <iostream>
   #include <fstream>
   #include <string>
   #include <algorithm>
   #include <cstdlib>
   #include <climits>
   #include <ctime>
   #include <list>
   using namespace std;
  
  #define TABU_SIZE 10     
  #define SWAPSIZE 5      
  #define ITERATIONS 100
  #define INF INT_MAX
 } 
  int rowIndex; 
  double adj[60][60];
  int ordered[60][60];
  int city1[60], city2[60], path[60];
  string filename[10] = {"gr17.tsp", "gr21.tsp", "gr24.tsp", "fri26.tsp", "bayg29.tsp", "bays29.tsp", "swiss42.tsp", "gr48.tsp", "hk48.tsp", "brazil58.tsp"};
  int bestans[10] = {2085, 2707, 1272, 937, 1610, 2020, 1273, 5046, 11461, 25395}; 
  int bestIteration;
  int tabuList[2000][4];
 
 
  bool cmp(int a, int b);
  double TabuSearch(const int & N);
  double GetPathLen(int* city, const int & N);
 
  int main(){
     string absolute("C:\\");
     int CASE;
     srand(time(0));
     while (cin >> CASE && CASE < 10 && CASE > -1){
         memset(adj, 0, sizeof(adj));
         memset(city1, 0, sizeof(city1));
         memset(city2, 0, sizeof(city2));
         memset(tabuList, 0, sizeof(tabuList));
         memset(path, 0, sizeof(path));
 
         string relative = filename[CASE];
         string filepath = absolute+relative;
         ifstream infile(filepath.c_str());
         if (infile.fail()){
             cout << "Open failed!\n";
         }
         int n;
         infile >> n;
         for (int j = 0; j < n; j++){
             for (int k = 0; k < n; k++){
                 infile >> adj[j][k];
            }
         }
 
         clock_t start, end;
         start = clock();
         int distance = TabuSearch(n);
         end = clock();
         double costTime = (end - start)*1.0/CLOCKS_PER_SEC;
         cout << "TSP file: " << filename[CASE] << endl;
         cout << "Optimal Soluton: " << bestans[CASE] << endl;
         cout << "Minimal distance: " << distance << endl;
         cout << "Error: " << (distance - bestans[CASE]) * 100 / bestans[CASE] << "%" << endl;
         cout << "Best iterations:  " << bestIteration << endl;
         cout << "Cost time:        " << costTime << endl; 
         cout << "Path:\n";
         for (int i = 0; i < n; i++){
             cout << path[i] + 1 << " ";
         }
         cout << endl << endl;;
         infile.close();
     }
     return 0;
 }
 
 
 
  void CreateRandOrder(int* city, const int & N){
     for (int i = 0; i < N; i++){
         city[i] = rand() % N;
         for (int j = 0; j < i; j++){
             if (city[i] == city[j]){
                 i--;
                 break;
             }
         }
     }
 }
 
 
  double GetPathLen(int* city, const int & N){
      double res = adj[city[N-1]][city[0]];
     int i;
     for (i = 1; i < N; i++){
         res += adj[city[i]][city[i-1]];
     }
     return res;
 }
 
 
 void UpdateTabuList(int len){
    for (int i = 0; i < len; i++){
        if (tabuList[i][3] > 0)
            tabuList[i][3]--;
    }
}


 double TabuSearch(const int & N){
    int countI, countN, NEIGHBOUR_SIZE = N * (N - 1) / 2;
    double bestDis, curDis, tmpDis, finalDis = INF;
    bestIteration = 0; 
    string bestCode, curCode, tmpCode;

    
    int i = 0;
    for (int j = 0; j < N - 1; j++){
        for (int k = j + 1; k < N; k++){
            tabuList[i][0] = j;
            tabuList[i][1] = k;
            tabuList[i][2] = INF;
            i++;
        }
    }


    
    for (int t = 0; t < 25; t++){
        CreateRandOrder(city1, N);
        bestDis = GetPathLen(city1, N);

    
        countI = ITERATIONS;
        int a, b;
        int pardon[2], curBest[2];
        while (countI--){
            countN = NEIGHBOUR_SIZE;
            pardon[0] = pardon[1] = curBest[0] = curBest[1] = INF;
            memcpy(city2, city1, sizeof(city2));
            //NEIGHBOUR_SIZE 
            while (countN--){
                
                a = tabuList[countN][0];
                b = tabuList[countN][1];
                swap(city2[a], city2[b]);
                tmpDis = GetPathLen(city2, N);
                 
                if (tabuList[countN][3] > 0){ 
                    tabuList[countN][2] = INF; 
                    if (tmpDis < pardon[1]){
                        pardon[0] = countN;
                        pardon[1] = tmpDis;
                    }
                }
                
                else {
                    tabuList[countN][2] = tmpDis;
                }
                swap(city2[a], city2[b]); 
            }
             
            for (int i = 0; i < NEIGHBOUR_SIZE; i++){
                if (tabuList[i][3] == 0 && tabuList[i][2] < curBest[1]){
                    curBest[0] = i;
                    curBest[1] = tabuList[i][2];
                }
            }
             
            if (curBest[0] == INF || pardon[1] < bestDis) {
                curBest[0] = pardon[0];
                curBest[1] = pardon[1];
           }

            
            if (curBest[1] < bestDis){
                bestDis = curBest[1];
                tabuList[curBest[0]][3] = TABU_SIZE;
                bestIteration = ITERATIONS - countI;
              a = tabuList[curBest[0]][0];
                b = tabuList[curBest[0]][1];
                swap(city1[a], city1[b]);
            }
            UpdateTabuList(NEIGHBOUR_SIZE);
        }
        
        if (bestDis < finalDis){
            finalDis = bestDis;
            memcpy(path, city1, sizeof(path));
       }
   }
    return finalDis;
}
