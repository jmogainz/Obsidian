Normalization
	*placeholder

IDEAS:
	Timeseries on 3 realtime seconds
	

AIMLPredictionServer.py
	Running two threads (two servers)
		One for classifying
		One for prioritizing
		
Current classes in config files
	5200 - Mortar bomb
	1122 - UAV SRECON
	1112 - UAV SLOW
	2200 - MISSLE
	1111 - UAV FAST
	
	
	
slow
MERGE NETWORK
	going slow - train longer, make bigger
	performing well on dev set still
	more data
quick
DENSE NETWORK
	beginning to overfit
	make bigger
	add regularization
	

Make network bigger
	Add batch norm and other regularization methods to decrease variance (overfitting)
Test only numericals
Test only matrices
Try with more epochs

CREATE MORE MODELS

ALL TESTS ON SAME DATA SET LATEST INUSE
	6000 per
######################################
128 batch size, recording dev set accuracy
First 7 num features (without covariance)

All
	50 epoch: .7522
		![[Pasted image 20220623135038.png]]
No SNR
	50 epoch: .4864
No velocity
	50 epoch: .6274
No altitude
	Did not make it: .3619 at epoch 32
No vertical velocity
	50 epoch: .7075
		![[Pasted image 20220623134813.png]]
No range
	50 epoch: .5619
No bearing
	50 epoch: .5654
No range-rate!!!! 
	early stopped: epoch 34: .2040
		![[Pasted image 20220623135736.png]]


########################################
128 batch size, recording dev set accuracy
State covariance matrix CNN
	- we do not know enough about covariance matrices
	- cannot normalize to uniform maximum like images
	- cannot relate one matrix to another
	
Normalized by internal max per samp
	50 epoch: .7338
		![[Pasted image 20220623141057.png]]
Normalized from matrix postion across samples
	50 epoch: .4771
		![[Pasted image 20220623141947.png]]


#########################################
128 batch size, recording dev set accuracy
Residual Covariance matrix CNN
	
Normalized by internal max per samp
	50 epoch: .5360
	
	
#########################################
128 batch size, recording dev set accuracy
State Covariance matrix Dense
	
Normalized by internal max per samp
	50 epoch: .5629
	
	
#########################################
128 batch size, recording dev set accuracy
Residual Covariance matrix Dense
	
Normalized by internal max per samp
	50 epoch: .3014
	
	
#########################################
128 batch size, recording dev set accuracy
Residual Covariance and State Covariance Combined Dense
	
Normalized by internal max per samp
	50 epoch: .5500
	
	
#########################################
128 batch size, recording dev set accuracy
All features combined Dense
	50 epoch: .8514
		![[Pasted image 20220623150747.png]]
		
		
#########################################
128 batch size, recording dev set accuracy
7 features Dense + SCOV CNN + RCOV CNN
	50 epoch: .7782
		![[Pasted image 20220623152836.png]]
		
		
#########################################
128 batch size, recording dev set accuracy
All features Dense + SCOV CNN
	50 epoch: .8597
		![[Pasted image 20220623154108.png]]
		
		
		
LATEST INUSE
	18000 per
	
	
#########################################
128 batch size, dev set accuracy
	Num, SCOV Dense
		50 epoch: .8970
		
		
#########################################
128 batch size, dev set accuracy
	All Dense
		50 epoch: .9041
		
		
#########################################
128 batch size, dev set accuracy
	Merge All Dense + SCOV CNN
		50 epoch: .9136
			![[Pasted image 20220624110550.png]]
			
			
			
