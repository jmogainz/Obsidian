Sigmoid
	Binary Class in Logistic-R (sum not to 1)
	Used as activation function
Softmax
	Multi Class in Logistic-R
	Sum will be 1
	Used on entire layers
Multi-Input Network
	concatenate vs. Merge
		concatenate - functional model (two layers)
		Merge - sequential model (two models)
Epochs
	Optimizer iterations (how many forward and back prop iterations followed by weight updates)
Batch_Size
	Split dataset into batches of a certain size
		Can speed up training speed (using less memory)
	Params are only updated after passing through the whole data set
		It must do forward and back prop and just store weight derivatives in a buffer
		Then when it is through each batch, it averages all the weight updates together
	Yes small batches do perform better...hypothetically if no batches are used, the entire dataset (all training samples) are forward propagated and then back propagated. Weight adjustments are determined for every single training sample, they are all averaged together, and then the weights are updated using the result. In batches, I assume the exact same thing occurs, except the entire dataset is just put through in chunks rather than all at once. At the end of every batch, the weight adjustments for each sample in said batch are stored in a buffer, and then when all batches are complete, the buffer of adjustments is averaged, and then the weights are updated with the result.
		However....this clearly must not be the case. If it was, mathematically, it would not affect the actual training / update of weights at each epoch (gradient descent pass). So what must be happening is the following: at the end of each batch, instead of weight adjustments being stored in the buffer, they must go ahead and average and update model weights. And then this occur every single time.....this would alter the way the model is trained...
Multivariate Time Series 
	Dataset = 2d array
Standardization VS. Normalization
	Standardization = mean: 0, std_dev: 1
		Do if distribution is normal
	Normalization = scale: 0-1
		Do if distribution is not normal
Hidden Layers
	2/3 size of input layer + output layer (lol, no way)
	Between input and output layer
	WHO KNOWS lol
When to Use CNN's
	https://www.quora.com/How-can-convolutional-neural-networks-be-used-for-non-image-data
Set Splits
	The larger your feature set, the more training samples you may need to fit your models with low bias.
	Y sets can be dataframes, x sets cannot
Outliers
	**Outliers should be removed from your dataset if you believe that the data point is incorrect or that the data point is so unrepresentative of the real world situation that it would cause your machine learning model to not generalise.**
LSTMS
	https://towardsdatascience.com/a-practical-guide-to-rnn-and-lstm-in-keras-980f176271bc
