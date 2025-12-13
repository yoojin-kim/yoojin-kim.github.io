---
title: "[D-74] CausalLM HEAD."  
categories: [Thesis]  
tags:
- [log, thesis]

toc: true
toc_sticky: true
date: 2025-12-13 12:00
---
## First Steps: Implementing BigsForCausalLM

Add a new class called `BiGSForCausalLM`
<details>
<summary> BiGSForCausalLM _ Python</summary>

```python
@add_start_docstrings("""BiGS Model with a **causal language modeling** head on top. """, BIGS_START_DOCSTRING)
class BiGSForCausalLM(BiGSPreTrainedModel):
    def __init__(self, config):
        super().__init__(config)

        # The core BiGS model (the encoder)
        self.bigs = BiGSModel(config)
        # The language modeling head (same structure as MLM head)
        self.lm_head = BiGSOnlyMLMHead(config)

        # Initialize weights and apply final processing
        self.post_init()

    def get_output_embeddings(self):
        return self.lm_head.predictions.decoder

    def set_output_embeddings(self, new_embeddings):
        self.lm_head.predictions.decoder = new_embeddings

    @add_start_docstrings_to_model_forward(BIGS_INPUTS_DOCSTRING.format("batch_size, sequence_length"))
    @add_code_sample_docstrings(
        processor_class=_TOKENIZER_FOR_DOC,
        checkpoint=_CHECKPOINT_FOR_DOC,
        output_type=MaskedLMOutput,
        config_class=_CONFIG_FOR_DOC,
    )
    def forward(
            self,
            input_ids=None,
            attention_mask=None,
            token_type_ids=None,
            position_ids=None,
            labels=None,
            output_hidden_states=None,
            return_dict=None,
    ):
        r"""
        labels (`torch.LongTensor` of shape `(batch_size, sequence_length)`, *optional*):
            Labels for computing the language modeling loss.
            Labels are shifted inside the model for causal modeling.
        """
        return_dict = return_dict if return_dict is not None else self.config.use_return_dict

        # 1. Pass input through the BiGS encoder
        outputs = self.bigs(
            input_ids,
            attention_mask=attention_mask,
            token_type_ids=token_type_ids,
            position_ids=position_ids,
            output_hidden_states=output_hidden_states,
            return_dict=return_dict,
        )

        sequence_output = outputs[0]
        # 2. Project hidden states to vocabulary size
        prediction_scores = self.lm_head(sequence_output)

        causal_lm_loss = None
        if labels is not None:
            # Shift the inputs and labels for Causal Language Modeling
            # This is the core difference for the loss calculation:
            # The model predicts the token at position i using the output at position i-1
            # In BiGS/BERT-style models, which are generally non-causal, we simplify
            # the training by predicting position i from the output at position i, but
            # ensuring that the loss is calculated against the *next* token (labels shifted).
            shifted_prediction_scores = prediction_scores[:, :-1, :].contiguous()
            shifted_labels = labels[:, 1:].contiguous()

            # 3. Compute loss
            loss_fct = CrossEntropyLoss()
            causal_lm_loss = loss_fct(shifted_prediction_scores.view(-1, self.config.vocab_size), shifted_labels.view(-1))

        if not return_dict:
            output = (prediction_scores,) + outputs[1:]
            return ((causal_lm_loss,) + output) if causal_lm_loss is not None else output

        return MaskedLMOutput( # Using MaskedLMOutput as it has the necessary fields (loss, logits, hidden_states)
            loss=causal_lm_loss,
            logits=prediction_scores,
            hidden_states=outputs.hidden_states,
        )
```

</details>

## Pre-training for "Fill in the Middle"

The "FIM" task is typically an *encoder-decoder* or *non-causal decoder* pre-training objective,
where the model is given the start and end of a text and must generate the missing middle part.

- Using `BiGSForCausalLM` for FIM: Since BiGS appears to be an *encoder-only* model
  (similar to BERT, judging by the `BiGSEncoder` and lack of a distinct decoder),
adapting it for a FIM task is often done by structuring the input as a single sequence with special tokens:

PREFIX + [FIM_MIDDLE] + SUFFIX -> MIDDLE + [FIM_END] + SUFFIX

- The training objective then becomes a standard *Causal Language Modeling*
task on this reordered sequence. By implementing `BiGSForCausalLM`,
you are providing the model structure needed to train on this sequence in 
an autoregressive(Causal) manner, forcing it to predict one token at a time.



