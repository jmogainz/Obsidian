400k ds / 100 estimators
	.947
	
700k ds / 1000 estimators
	.982
	
700k ds / GRID SEARCHED OPTIMAL / Suboptimal outlier removal
-Best: 0.962727 using {'learning_rate': 0.3, 'max_depth': 6, 'n_estimators': 3500}
	.985 on 109k test samples
	.995 on 200..k test samples
	
90k ds / 100 estimators / Suboptimal outlier removal 
- SNR included
	.9569 on 26k test samples
- SNR excluded
	.9525 on 26k test samples
	
# conclusion: SNR is useful for all besides KNN; however, RESIDUAL BLOWS, only benefits NN	

1.4 m / 3500 estimators / GRID SEARCHED OPTIMAL / NO outlier removal
-
Clean Dataset, recorded no outliers, and 6 Classes
-
Acc: .9839 on 426k samples
	![[Pasted image 20220721092312.png]]
