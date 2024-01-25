https://chat.openai.com/share/e5b58afb-fad6-4469-b207-fbb7cd2fd6f3
https://chat.openai.com/share/61618e09-a66f-458c-a9b2-4afad1ad021e

- Is attention the defining reason why the original transformer was capable of learning sequential data. It seems that without recurrency, there has to be attention between the encoder and decoder to learn relationships and patters from one time step to the next?
- Does the TFT model use recurrency?
- Encoder-decoder allows for seq-to-seq learning. The entire sequence is first processed before the decoder runs off of the final hidden state of the encoder. The decoder is then told how many times to iterate to output a variable length sequence. 
- Self-attention differs from dot-product attention (which takes place between the decoder and encoder), what is it?
- With these advanced sequence-to-sequence models, how are they trained? Are there labels (truth output sequences)? How is loss calculated? How does back prop work?
- TFT specifically is great at time series forecasting for its ability to handle time-variant and static exogenous variables (many types of features). 
![[Pasted image 20230704145131.png]]
	- number of sales is *target variable*
	- Consumer Price Index or number of visitors are *time-varying unknown*
	- holidays and special days are *time-varying known* events
	- product id is a *time-invariant categorical*
	- yearly revenue is *time-invariant real*
- 