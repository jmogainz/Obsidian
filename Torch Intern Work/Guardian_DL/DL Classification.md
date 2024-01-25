- ONE Neuron, (0:1111, .25:1112, .5:1122, .75:2200, 1:5200)
	- regression task ^
- Multi-Neuron: simpler and more effective

Multi-Neuron, 128 batch, early stop patience 80
	Dense network with dropouts eliminates slight overfitting on 90k dataset
	epoch 350: .9268
	POSSIBLY: Train longer with more patience
		Todo: We need MORE data, and bigger network
			If starts to overfit, 
	![[Pasted image 20220628061617.png]]
	![[Pasted image 20220628061633.png]]
	-
	220k ds
	epoch 350: .9326
	POSSIBLY: Train longer with more patience
		Todo: More data, bigger network
	![[Pasted image 20220628141316.png]]
	![[Pasted image 20220628141328.png]]
	-
	440k ds
	epoch 642: .9337
	![[Pasted image 20220630083601.png]]
	![[Pasted image 20220630083625.png]]
		-
		lr -> .0005
		epoch 451: .9358
		![[Pasted image 20220630120002.png]]
		![[Pasted image 20220630120022.png]]
	
	
Multi-Neuron, 128 batch, early stop patience 150
	Larger Dense Network with dropouts
	200k ds epoch 800: .9298
	NOTE: Larger clearly not always better
	![[Pasted image 20220629070032.png]]
	![[Pasted image 20220629070050.png]]
	
TODO: elminate all range outliers
	
	