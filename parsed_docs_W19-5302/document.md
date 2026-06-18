<span id="page-0-0"></span>0
# Results of the WMT19 Metrics Shared Task: Segment-Level and Strong MT Systems Pose Big Challenges

Qingsong Ma Tencent-CSIG, AI Evaluation Lab qingsong.mqs@gmail.com

Ondřej Bojar Charles University, MFF ÚFAL bojar@ufal.mff.cuni.cz

Johnny Tian-Zheng Wei UMass Amherst, CICS jwei@umass.edu

Yvette Graham Dublin City University, ADAPT graham.yvette@gmail.com

## Abstract

This paper presents the results of the WMT19 Metrics Shared Task. Participants were asked to score the outputs of the translations systems competing in the WMT19 News Translation Task with automatic metrics. 13 research groups submitted 24 metrics, 10 of which are reference-less “metrics” and constitute submissions to the joint task with WMT19 Quality Estimation Task, “QE as a Metric”. In addition, we computed 11 baseline metrics, with 8 commonly applied baselines (BLEU, SentBLEU, NIST, WER, PER, TER, CDER, and chrF) and 3 reimplementations (chrF+, sacreBLEU-BLEU, and sacreBLEU-chrF). Metrics were evaluated on the system level, how well a given metric correlates with the WMT19 official manual ranking, and segment level, how well the metric correlates with human judgements of segment quality. This year, we use direct assessment (DA) as our only form of manual evaluation.

## 1 Introduction

To determine system performance in machine translation (MT), it is often more practical to use an automatic evaluation, rather than a manual one. Manual/human evaluation can be costly and time consuming, and so an automatic evaluation metric, given that it sufficiently correlates with manual evaluation, can be useful in developmental cycles. In studies involving hyperparameter tuning or architecture search, automatic metrics are necessary as the amount of human effort implicated in manual evaluation is generally prohibitively large. As objective, reproducible quantities, metrics can also facilitate cross-paper comparisons. The WMT Metrics Shared Task1 annually serves as a venue to validate the use of existing metrics (including baselines such as BLEU), and to develop new ones; see Koehn and Monz (2006) through Ma et al. (2018).

In the setup of our Metrics Shared Task, an automatic metric compares an MT system’s output translations with manual reference translations to produce: either (a) system-level score, i.e. a single overall score for the given MT system, or (b) segment-level scores for each of the output translations, or both.

This year we teamed up with the organizers of the QE Task and hosted “QE as a Metric” as a joint task. In the setup of the Quality Estimation Task (Fonseca et al., 2019), no humanproduced translations are provided to estimate the quality of output translations. Quality estimation (QE) methods are built to assess MT output based on the source or based on the translation itself. In this task, QE developers were invited to perform the same scoring as standard metrics participants, with the exception that they refrain from using a reference translation in production of their scores. We then evaluate the QE submissions in exactly the same way as regular metrics are evaluated, see below. From the point of view of correlation with manual judgements, there is no difference in metrics using or not using references.

The source, reference texts, and MT system outputs for the Metrics task come from the News Translation Task (Barrault et al., 2019, which we denote as Findings 2019). The texts were drawn from the news domain and involve translations of English (en) to/from

<span id="page-1-0"></span>1
Czech (cs), German (de), Finnish (fi), Gujarati (gu), Kazakh (kk), Lithuanian (lt), Russian (ru), and Chinese (zh), but excluding csen (15 language pairs). Three other language pairs not including English were also manually evaluated as part of the News Translation Task: German Czech and German French. In total, metrics could participate in 18 language pairs, with 10 target languages.

In the following, we first give an overview of the task (Section 2) and summarize the baseline (Section 3) and submitted (Section 4) metrics. The results for system- and segment-level evaluation are provided in Sections 5.1 and 5.2, respectively, followed by a joint discussion Section 6.

## 2 Task Setup

This year, we provided task participants with one test set for each examined language pair, i.e. a set of source texts (which are commonly ignored by MT metrics), corresponding MT outputs (these are the key inputs to be scored) and a reference translation (held out for the participants of “QE as a Metric” track).

In the system-level, metrics aim to correlate with a system’s score which is an average over many human judgments of segment translation quality produced by the given system. In the segment-level, metrics aim to produce scores that correlate best with a human ranking judgment of two output translations for a given source segment (more on the manual quality assessment in Section 2.3). Participants were free to choose which language pairs and tracks (system/segment and reference-based/reference-free) they wanted to take part in.

## 2.1 Source and Reference Texts

The source and reference texts we use are newstest2019 from this year’s WMT News Translation Task (see Findings 2019). This set contains approximately 2,000 sentences for each translation direction (except Gujarati, Kazakh and Lithuanian which have approximately 1,000 sentences each, and German to/from French which has 1701 sentences).

The reference translations provided in newstest2019 were created in the same direction as the MT systems were translating.

The exceptions are German Czech where both sides are translations from English and German French which followed last years’ practice. Last year and the years before, the dataset consisted of two halves, one originating in the source language and one in the target language. This however lead to adverse artifacts in MT evaluation.

## 2.2 System Outputs

The results of the Metrics Task are affected by the actual set of MT systems participating in a given translation direction. On one hand, if all systems are very close in their translation quality, then even humans will struggle to rank them. This in turn will make the task for MT metrics very hard. On the other hand, if the task includes a wide range of systems of varying quality, correlating with humans should be generally easier, see Section 6.1 for a discussion on this. One can also expect that if the evaluated systems are of different types, they will exhibit different error patterns and various MT metrics can be differently sensitive to these patterns.

This year, all MT systems included in the Metrics Task come from the News Translation Task (see Findings 2019). There are however still noticeable differences among the various language pairs.

• Unsupervised MT Systems. The German→Czech research systems were trained in an unsupervised fashion, i.e. without the access to parallel Czech-German texts (except for a couple of thousand sentences used primarily for validation). We thus expect the research German-Czech systems to be “more creative” and depart further away from the references. The online systems in this language directions are however standard MT systems so the German-Czech evaluation could be to some extent bimodal.

• EU Election. The French German translation was focused on a sub-domain of news, namely texts related EU Election. Various MT system developers may have invested more or less time to the domain adaptation.

• Regular News Tasks Systems. These are all the other MT systems in the evaluation; differing in whether they are trained only on WMT provided data (“Constrained”, or “Unconstrained”) as in the previous years. All the freely available web services (online MT systems) are deemed unconstrained.

<span id="page-2-0"></span>2
Overall, the results are based on 233 systems across 18 language pairs.2

## 2.3 Manual Quality Assessment

Direct Assessment (DA, Graham et al., 2013, 2014a, 2016) was employed as the source of the “golden truth” to evaluate metrics again this year. The details of this method of human evaluation are provided in Findings 2019.

The basis of DA is to collect a large number of quality assessments (a number on a scale of 1–100, i.e. effectively a continuous scale) for the outputs of all MT systems. These scores are then standardized per annotator.

In the past years, the underlying manual scores were reference-based (human judges had access to the same reference translation as the MT quality metric). This year, the official WMT19 scores are reference-based (or “monolingual”) for some language pairs and reference-free (or “bilingual”) for others.3

Due to these different types of golden truth collection, reference-based language pairs are in a closer match with the standard referencebased metrics, while the reference-free language pairs are better fit for the “QE as a metric” subtask.

Note that system-level manual scores are different than those of the segment-level. Since for segment-level evaluation, collecting enough DA judgements for each segment is infeasible, so we resort to converting DA judgements to golden truth expressed as relative rankings, see Section 2.3.2.

The exact methods used to calculate correlations of participating metrics with the golden truth are described below, in the two sections for system-level evaluation (Section 5.1) and segment-level evaluation (Section 5.2).

## 2.3.1 System-level Golden Truth: DA

For the system-level evaluation, the collected continuous DA scores, standardized for each annotator, are averaged across all assessed segments for each MT system to produce a scalar rating for the system’s performance.

The underlying set of assessed segments is different for each system. Thanks to the fact that the system-level DA score is an average over many judgments, mean scores are consistent and have been found to be reproducible (Graham et al., 2013). For more details see Findings 2019.

## 2.3.2 Segment-level Golden Truth: daRR

Starting from Bojar et al. (2017), when WMT fully switched to DA, we had to come up with a solid golden standard for segment-level judgements. Standard DA scores are reliable only when averaged over sufficient number of judgments.4

Fortunately, when we have at least two DA scores for translations of the same source input, it is possible to convert those DA scores into a relative ranking judgement, if the difference in DA scores allows conclusion that one translation is better than the other. In the following, we denote these re-interpreted DA judgements as “daRR”, to distinguish it clearly from the relative ranking (“RR”) golden truth used in the past years.5

<span id="page-3-0"></span>3
Table 1: Number of judgements for DA converted to daRR data; “DA>1” is the number of source input sentences in the manual evaluation where at least two translations of that same source input segment received a DA judgement; “Ave” is the average number of translations with at least one DA judgement available for the same source input sentence; “DA pairs” is the number of all possible pairs of translations of the same source input resulting from “DA>1”; and “daRR” is the number of DA pairs with an absolute difference in DA scores greater than the 25 percentage point margin.
![](tables/table_pg3_num0.csv)

From the complete set of human assessments collected for the News Translation Task, all possible pairs of DA judgements attributed to distinct translations of the same source were converted into daRR better/worse judgements. Distinct translations of the same source input whose DA scores fell within 25 percentage points (which could have been deemed equal quality) were omitted from the evaluation of segment-level metrics. Conversion of scores in this way produced a large set of daRR judgements for all language pairs, shown in Table 1 due to combinatorial advantage of extracting daRR judgements from all possible pairs of translations of the same source input. We see that only German-French and esp. French-German can suffer from insufficient number of these simulated pairwise comparisons.

The daRR judgements serve as the golden standard for segment-level evaluation in WMT19.

## 3 Baseline Metrics

In addition to validating popular metrics, including baselines metrics serves as comparison and prevents “loss of knowledge” as mentioned by Bojar et al. (2016).

Moses scorer6 is one of the MT evaluation tools that aggregated several useful metrics over the time. Since Macháček and Bojar (2013), we have been using Moses scorer to provide most of the baseline metrics and kept encouraging authors of well-performing MT metrics to include them in Moses scorer.7

The baselines we report are:

BLEU and NIST The metrics BLEU (Papineni et al., 2002) and NIST (Doddington, 2002) were computed using mteval-v13a.pl8 from the OpenMT Evaluation Campaign. The tool includes its own tokenization. We run mteval with the flag --international-tokenization.9

TER, WER, PER and CDER. The metrics TER (Snover et al., 2006), WER, PER and CDER (Leusch et al., 2006) were produced by the Moses scorer, which is used in Moses model optimization. We used the standard tokenizer script as available in Moses toolkit for tokenization.

sentBLEU. The metric sentBLEU is computed using the script sentence-bleu, a part of the Moses toolkit. It is a smoothed version of BLEU for scoring at the segment-level. We used the standard tokenizer script as available in Moses toolkit for tokenization.

<span id="page-4-0"></span>4
MT19MetricsSharedTask“•”denotesthatthemetrictookpartinsomeofthelanguagepairs)oftheseg .         (       hat the system-level scores are implied, simply taking arithmetic (macro-)average of segment-level scores. “ − ) k (Seg/Sys-level. A metric is learned if it is trained on a QE or metric evaluation dataset (i.e. pretraining o ) etrics task data does. For the baseline metrics available in the Moses toolkit, paths are relative to http:/ sesdecoder
![](tables/table_pg4_num0.csv)

<span id="page-5-0"></span>5
chrF and chrF+. The metrics chrF and chrF+ (Popović, 2015, 2017) are computed using their original Python implementation, see Table 2. We ran chrF++.py with the parameters -nw 0 -b 3 to obtain the chrF score and with -nw 1 -b 3 to obtain the chrF+ score. Note that chrF intentionally removes all spaces before matching the n-grams, detokenizing the segments but also concatenating words.10

sacreBLEU-BLEU and sacreBLEUchrF. The metrics sacreBLEU-BLEU and sacreBLEU-chrF (Post, 2018a) are re-implementation of BLEU and chrF respectively. We ran sacreBLEU-chrF with the same parameters as chrF, but their scores are slightly different. The signature strings produced by sacreBLEU for BLEU and chrF respectively are BLEU+case.lc+lang.de-en+numrefs.1+ smooth.exp+tok.intl+version.1.3.6 and chrF3+case.mixed+lang.de-en +numchars.6+numrefs.1+space.False+ tok.13a+version.1.3.6.

The baselines serve in system and segmentlevel evaluations as customary: BLEU, TER, WER, PER, CDER, sacreBLEU-BLEU and sacreBLEU-chrF for system-level only; sentBLEU for segment-level only and chrF for both.

Chinese word segmentation is unfortunately not supported by the tokenization scripts mentioned above. For scoring Chinese with baseline metrics, we thus pre-processed MT outputs and reference translations with the script tokenizeChinese.py11 by Shujian Huang, which separates Chinese characters from each other and also from non-Chinese parts.

## 4 Submitted Metrics

Table 2 lists the participants of the WMT19 Shared Metrics Task, along with their metrics and links to the source code where available. We have collected 24 metrics from a total of 13 research groups, with 10 reference-less “metrics” submitted to the joint task “QE as a Metrich” with WMT19 Quality Estimation Task.

The rest of this section provides a brief summary of all the metrics that participated.

## 4.1 BEER

BEER (Stanojević and Sima’an, 2015) is a trained evaluation metric with a linear model that combines sub-word feature indicators (character n-grams) and global word order features (skip bigrams) to achieve a language agnostic and fast to compute evaluation metric. BEER has participated in previous years of the evaluation task.

## 4.2 BERTr

BERTr (Mathur et al., 2019) uses contextual word embeddings to compare the MT output with the reference translation.

The BERTr score of a translation is the average recall score over all tokens, using a relaxed version of token matching based on BERT embeddings: namely, computing the maximum cosine similarity between the embedding of a reference token against any token in the MT output. BERTr uses bert\_base\_uncased embeddings for the to-English language pairs, and bert\_base\_multilingual\_cased embeddings for all other language pairs.

## 4.3 CharacTER

CharacTER (Wang et al., 2016b,a), identical to the 2016 setup, is a character-level metric inspired by the commonly applied translation edit rate (TER). It is defined as the minimum number of character edits required to adjust a hypothesis, until it completely matches the reference, normalized by the length of the hypothesis sentence. CharacTER calculates the character-level edit distance while performing the shift edit on word level. Unlike the strict matching criterion in TER, a hypothesis word is considered to match a reference word and could be shifted, if the edit distance between them is below a threshold value. The Levenshtein distance between the reference and the shifted hypothesis sequence is computed on the character level. In addition, the lengths of hypothesis sequences instead of reference sequences are used for normalizing the edit distance, which effectively counters the issue that shorter translations normally achieve lower TER.

<span id="page-6-0"></span>6
Similarly to other character-level metrics, CharacTER is generally applied to nontokenized outputs and references, which also holds for this year’s submission with one exception. This year tokenization was carried out for en-ru hypotheses and references before calculating the scores, since this results in large improvements in terms of correlations. For other language pairs, no tokenizer was used for pre-processing.

## 4.4 EED

EED (Stanchev et al., 2019) is a characterbased metric, which builds upon CDER. It is defined as the minimum number of operations of an extension to the conventional edit distance containing a “jump” operation. The edit distance operations (insertions, deletions and substitutions) are performed at the character level and jumps are performed when a blank space is reached. Furthermore, the coverage of multiple characters in the hypothesis is penalised by the introduction of a coverage penalty. The sum of the length of the reference and the coverage penalty is used as the normalisation term.

## 4.5 ESIM

Enhanced Sequential Inference Model (ESIM; Chen et al., 2017; Mathur et al., 2019) is a neural model proposed for Natural Language Inference that has been adapted for MT evaluation. It uses cross-sentence attention and sentence matching heuristics to generate a representation of the translation and the reference, which is fed to a feedforward regressor. The metric is trained on singly-annotated Direct Assessment data that has been collected for evaluating WMT systems: all WMT 2018 to-English data for the to-English language pairs, and all WMT 2018 data for all other language pairs.

## 4.6 hLEPORb\_baseline, hLEPORa\_baseline

The submitted metric hLEPOR\_baseline is a metric based on the factor combination of length penalty, precision, recall, and position difference penalty. The weighted harmonic mean is applied to group the factors together with tunable weight parameters. The systemlevel score is calculated with the same formula but with each factor weighted using weight estimated at system-level and not at segmentlevel.

In this submitted baseline version, hLE-POR\_baseline was not tuned for each language pair separately but the default weights were applied across all submitted language pairs. Further improvements can be achieved by tuning the weights according to the development data, adding morphological information and applying n-gram factor scores into it (e.g. part-of-speech, n-gram precision and n-gram recall that were added into LEPOR in WMT13.). The basic model factors and further development with parameters setting were described in the paper (Han et al., 2012) and (Han et al., 2013).

For sentence-level score, only hLE-PORa\_baseline was submitted with scores calculated as the weighted harmonic mean of all the designed factors using default parameters.

For system-level score, both hLEPORa\_baseline and hLE-PORb\_baseline were submitted, where hLEPORa\_baseline is the the average score of all sentence-level scores, and hLE-PORb\_baseline is calculated via the same sentence-level hLEPOR equation but replacing each factor value with its system-level counterpart.

## 4.7 Meteor++\_2.0 (syntax), Meteor++\_2.0 (syntax+copy)

Meteor++ 2.0 (Guo and Hu, 2019) is a metric based on Meteor (Denkowski and Lavie, 2014) that takes syntactic-level paraphrase knowledge into consideration, where paraphrases may sometimes be skip-grams. i.e. (protect...from, protect...against). As the original Meteor-based metrics only pay attention to consecutive string matching, they perform badly when reference-hypothesis pairs contain skip n-gram paraphrases. Meteor++ 2.0 extracts the knowledge from the Paraphrase Database (PPDB; Bannard and Callison-Burch, 2005) and integrates it into Meteor-based metrics.

<span id="page-7-0"></span>7
## 4.8 PReP

PReP (Yoshimura et al., 2019) is a method for filtering pseudo-references to achieve a good match with a gold reference.

At the beginning, the source sentence is translated with some off-the-shelf MT systems to create a set of pseudo-references. (Here the MT systems were Google Translate and Microsoft Bing Translator.) The pseudoreferences are then filtered using BERT (Devlin et al., 2019) fine-tuned on the MPRC corpus (Dolan and Brockett, 2005), estimating the probability of the paraphrase between gold reference and pseudo-references. Thanks to the high quality of the underlying MT systems, a large portion of their outputs is indeed considered as a valid paraphrase.

The final metric score is calculated simply with SentBLEU with these multiple references.

## 4.9 WMDO

WMDO (Chow et al., 2019b) is a metric based on distance between distributions in the semantic vector space. Matching in the semantic space has been investigated for translation evaluation, but the constraints of a translation’s word order have not been fully explored. Building on the Word Mover’s Distance metric and various word embeddings, WMDO introduces a fragmentation penalty to account for fluency of a translation. This word order extension is shown to perform better than standard WMD, with promising results against other types of metrics.

## 4.10 YiSi-0, YiSi-1, YiSi-1\_srl, YiSi-2, YiSi-2\_srl

YiSi (Lo, 2019) is a unified semantic MT quality evaluation and estimation metric for languages with different levels of available resources.

YiSi-1 is a MT evaluation metric that measures the semantic similarity between a machine translation and human references by aggregating the idf-weighted lexical semantic similarities based on the contextual embeddings extracted from BERT and optionally incorporating shallow semantic structures (denoted as YiSi-1\_srl).

YiSi-0 is the degenerate version of YiSi-1 that is ready-to-deploy to any language. It uses longest common character substring to measure the lexical similarity.

YiSi-2 is the bilingual, reference-less version for MT quality estimation, which uses the contextual embeddings extracted from BERT to evaluate the crosslingual lexical semantic similarity between the input and MT output. Like YiSi-1, YiSi-2 can exploit shallow semantic structures as well (denoted as YiSi-2\_srl).

## 4.11 QE Systems

In addition to the submitted standard metrics, 10 quality estimation systems were submitted to the “QE as a Metric” track. The submitted QE systems are evaluated in the same settings as metrics to facilitate comparison. Their descriptions can be found in the Findings of the WMT 2019 Shared Task on Quality Estimation (Fonseca et al., 2019).

## 5 Results

We discuss system-level results for news task systems in Section 5.1. The segment-level results are in Section 5.2.

## 5.1 System-Level Evaluation

As in previous years, we employ the Pearson correlation (r) as the main evaluation measure for system-level metrics. The Pearson correlation is as follows:

$$
r = \frac { \sum _ { i = 1 } ^ { n } ( H _ { i } - \overline { { H } } ) ( M _ { i } - \overline { { M } } ) } { \sqrt { \sum _ { i = 1 } ^ { n } ( H _ { i } - \overline { { H } } ) ^ { 2 } } \sqrt { \sum _ { i = 1 } ^ { n } ( M _ { i } - \overline { { M } } ) ^ { 2 } } }\tag{1}
$$

where $H _ { i }$ are human assessment scores of all systems in a given translation direction, Mi are the corresponding scores as predicted by a given metric. H and M are their means, respectively.

Since some metrics, such as BLEU, aim to achieve a strong positive correlation with human assessment, while error metrics, such as TER, aim for a strong negative correlation we compare metrics via the absolute value |r| of a given metric’s correlation with human assessment.

<span id="page-8-0"></span>8
Table 3: Absolute Pearson correlation of to-English system-level metrics with DA human assessment in newstest2019; correlations of metrics not significantly outperformed by any other for that language pair are highlighted in bold.
![](tables/table_pg8_num0.csv)

<span id="page-9-0"></span>9
Table 4: Absolute Pearson correlation of out-of-English system-level metrics with DA human assessment in newstest2019; correlations of metrics not significantly outperformed by any other for that language pair are highlighted in bold.
![](tables/table_pg9_num0.csv)

<span id="page-11-0"></span>11
## 5.1.1 System-Level Results

Tables 3, 4 and 5 provide the system-level correlations of metrics evaluating translation of newstest2019. The underlying texts are part of the WMT19 News Translation test set (newstest2019) and the underlying MT systems are all MT systems participating in the WMT19 News Translation Task.

As recommended by Graham and Baldwin (2014), we employ Williams significance test (Williams, 1959) to identify differences in correlation that are statistically significant. Williams test is a test of significance of a difference in dependent correlations and therefore suitable for evaluation of metrics. Correlations not significantly outperformed by any other metric for the given language pair are highlighted in bold in Tables 3, 4 and 5.

Since pairwise comparisons of metrics may be also of interest, e.g. to learn which metrics significantly outperform the most widely employed metric BLEU, we include significance test results for every competing pair of metrics including our baseline metrics in Figure 1 and Figure 2.

This year, the increased number of systems participating in the news tasks has provided a larger sample of system scores for testing metrics. Since we already have sufficiently conclusive results on genuine MT systems, we do not need to generate hybrid system results as in Graham and Liu (2016) and past metrics tasks.

## 5.2 Segment-Level Evaluation

Segment-level evaluation relies on the manual judgements collected in the News Translation Task evaluation. This year, again we were unable to follow the methodology outlined in Graham et al. (2015) for evaluation of segment-level metrics because the sampling of sentences did not provide sufficient number of assessments of the same segment. We therefore convert pairs of DA scores for competing translations to daRR better/worse preferences as described in Section 2.3.2.

We measure the quality of metrics’ segmentlevel scores against the daRR golden truth using a Kendall’s Tau-like formulation, which is an adaptation of the conventional Kendall’s Tau coefficient. Since we do not have a total order ranking of all translations, it is not possible to apply conventional Kendall’s Tau (Graham et al., 2015).

Our Kendall’s Tau-like formulation, τ , is as follows:

$$
\tau = { \frac { | C o n c o r d a n t | - | D i s c o r d a n t | } { | C o n c o r d a n t | + | D i s c o r d a n t | } }\tag{2}
$$

where Concordant is the set of all human comparisons for which a given metric suggests the same order and Discordant is the set of all human comparisons for which a given metric disagrees. The formula is not specific with respect to ties, i.e. cases where the annotation says that the two outputs are equally good.

The way in which ties (both in human and metric judgement) were incorporated in computing Kendall τ has changed across the years of WMT Metrics Tasks. Here we adopt the version used in WMT17 daRR evaluation. For a detailed discussion on other options, see also Macháček and Bojar (2014).

Whether or not a given comparison of a pair of distinct translations of the same source input, s1 and s2, is counted as a concordant (Conc) or disconcordant (Disc) pair is defined by the following matrix:

![](tables/table_pg11_num0.csv)

In the notation of Macháček and Bojar (2014), this corresponds to the setup used in WMT12 (with a different underlying method of manual judgements, RR):

![](tables/table_pg11_num1.csv)

The key differences between the evaluation used in WMT14–WMT16 and evaluation used in WMT17–WMT19 were (1) the move from RR to daRR and (2) the treatment of ties. In the years 2014-2016, ties in metrics scores were not penalized. With the move to daRR, where the quality of the two candidate translations is deemed substantially different and no ties in human judgements arise, it makes sense to penalize ties in metrics’ predictions in order to promote discerning metrics.

<span id="page-12-0"></span>12

### Full Page Description (Page 12)

**Source:** `assets/_page_12_Asset_0.jpg`

**Generated:** 2026-05-30 21:55:52

---

The image contains a series of heatmaps, each representing the correlation between different metrics for machine translation quality evaluation. The heatmaps are labeled with "de-cs," "de-fr," and "fr-de," indicating the language pairs being evaluated. Each heatmap uses a color gradient to represent the correlation strength between metrics, with darker colors indicating a stronger correlation. The metrics listed on the axes include various evaluation scores such as BLEU, CIDER, WER, and others. The heatmaps show that some metrics are highly correlated with each other, while others have weaker correlations.

Table 5: Absolute Pearson correlation of system-level metrics for language pairs not involving English with DA human assessment in newstest2019; correlations of metrics not significantly outperformed by any other for that language pair are highlighted in bold.
![](tables/table_pg12_num0.csv)

<span id="page-13-0"></span>13
Table 6: Segment-level metric results for to-English language pairs in newstest2019: absolute Kendall’s Tau formulation of segment-level metric scores with DA scores; correlations of metrics not significantly outperformed by any other for that language pair are highlighted in bold.
![](tables/table_pg13_num0.csv)

<span id="page-14-0"></span>14
Table 7: Segment-level metric results for out-of-English language pairs in newstest2019: absolute Kendall’s Tau formulation of segment-level metric scores with DA scores; correlations of metrics not significantly outperformed by any other for that language pair are highlighted in bold.
![](tables/table_pg14_num0.csv)

Table 8: Segment-level metric results for language pairs not involving English in newstest2019: absolute Kendall’s Tau formulation of segment-level metric scores with DA scores; correlations of metrics not significantly outperformed by any other for that language pair are highlighted in bold.
![](tables/table_pg14_num1.csv)

Note that the penalization of ties makes our evaluation asymmetric, dependent on whether the metric predicted the tie for a pair where humans predicted <, or >. It is now important to interpret the meaning of the comparison identically for humans and metrics. For error metrics, we thus reverse the sign of the metric score prior to the comparison with human scores: higher scores have to indicate better translation quality. In WMT19, the original authors did this for CharacTER.

To summarize, the WMT19 Metrics Task for segment-level evaluation:

• ensures that error metrics are first converted to the same orientation as the human judgements, i.e. higher score indicating higher translation quality,

• excludes all human ties (this is already implied by the construction of daRR from DA judgements),

<span id="page-16-0"></span>16

### Full Page Description (Page 16)

**Source:** `assets/_page_16_Asset_0.jpg`

**Generated:** 2026-05-30 21:52:07

---

The image contains a series of heatmaps, each representing the performance of different translation models on three language pairs: de-cs, de-fr, and fr-de. The heatmaps display the correlation between various metrics and models, with green squares indicating a positive correlation. The x-axis lists the models, and the y-axis lists the metrics. The heatmaps show that for each language pair, certain models have higher correlations with specific metrics, suggesting a stronger relationship between those metrics and models.

• counts metric’s ties as a Discordant pairs.

We employ bootstrap resampling (Koehn, 2004; Graham et al., 2014b) to estimate confidence intervals for our Kendall’s Tau formulation, and metrics with non-overlapping 95% confidence intervals are identified as having statistically significant difference in performance.

## 5.2.1 Segment-Level Results

Results of the segment-level human evaluation for translations sampled from the News Translation Task are shown in Tables 6, 7 and 8, where metric correlations not significantly outperformed by any other metric are highlighted in bold. Head-to-head significance test results for differences in metric performance are included in Figures 3 and 4.

## 6 Discussion

This year, human data was collected from reference-based evaluations (or “monolingual”) and reference-free evaluations (or “bilingual”). The reference-based (monolingual) evaluations were obtained with the help of anonymous crowdsourcing, while the reference-less (bilingual) evaluations were mainly from MT researchers who committed their time contribution to the manual evaluation for each submitted system.

## 6.1 Stability across MT Systems

The observed performance of metrics depends on the underlying texts and systems that participate in the News Translation Task (see Section 2). For the strongest MT systems, distinguishing which system outputs are better is hard, even for human assessors. On the other hand, if the systems are spread across a wide performance range, it will be easier for metrics to correlate with human judgements.

Figure 5: Pearson correlations of sacreBLEU-BLEU for English-German system-level evaluation for all systems (left) down to only top 4 systems (right). The y-axis spans from -1 to +1, baseline metrics for the language pair in grey.

To provide a more reliable view, we created plots of Pearson correlation when the underlying set of MT systems is reduced to top n ones. One sample such plot is in Figure 5, all language pairs and most of the metrics are in Appendix A.

As the plot documents, the official correlations reported in Tables 3 to 5 can lead to wrong conclusions. sacreBLEU-BLEU correlates at .969 when all systems are considered, but as we start considering only the top n systems, the correlation falls relatively quickly. With 10 systems, we are below .5 and when only the top 6 or 4 systems are considered, the correlation falls even to the negave values. Note that correlations point estimates (the value in the y-axis) become noiser with the decreasing number of the underlying MT systems.

Figure 6 explains the situation and illustrates the sensitivity of the observed correlations to the exact set of systems. On the full set of systems, the single outlier (the worstperforming system called en\_de\_task) helps to achieve a great positive correlation. The majority of MT systems however form a cloud with Pearson correlation around .5 and the top 4 systems actually exhibit a negative correlation of the human score and sacreBLEU-BLEU.

<span id="page-17-0"></span>17

### Full Page Description (Page 17)

**Source:** `assets/_page_17_Asset_0.jpg`

**Generated:** 2026-05-30 21:55:19

---

The image is a scatter plot with a title "Figure 6". It displays data points representing different systems, categorized by their performance on SacreBLEU-BLEU metric and diversity (DA). The x-axis represents the diversity (DA) and the y-axis represents the SacreBLEU-BLEU metric. The plot includes trend lines for top 4, top 6, top 8, top 10, top 12, and top 15 systems, as well as a line representing all systems. The data points are color-coded and grouped by their performance on the SacreBLEU-BLEU metric. The trend lines show a general positive correlation between diversity and SacreBLEU-BLEU metric, with the top-performing systems generally falling on or near the top trend lines.

In Appendix A, baseline metrics are plotted in grey in all the plots, so that their trends can be observed jointly. In general, most baselines have similar correlations, as most baselines use similar features (n-gram or word-level features, with the exception of chrF). In a number of language pairs (de-en, de-fr, en-de, en-kk, lten, ru-en, zh-en), baseline correlations tend towards 0 (no correlation) or even negative Pearson correlation. For a widely applied metric such as sacreBLEU-BLEU, our analysis reveals weak correlation in comparing top stateof-the-art systems in these language pairs, especially in en-de, de-en, ru-en, and zh-en.

We will restrict our analysis to those language pairs where the baseline metrics have an obvious downward trend (de-en, de-fr, en-de, en-kk, lt-en, ru-en, zh-en). Examining the topn correlation in the submitted metrics (not including QE systems), most metrics show the same degredation in correlation as the baselines. We note BERTr as the one exception consistently degrading less and retaining positive correlation compared to other submitted metrics and baselines, in the language pairs where it participated.

For QE systems, we noticed that in some instances, QE systems have upward correlation trends when other metrics and baselines have downward trends. For instance, LP, UNI, and UNI+ in the de-en language pair, YiSi-2 in en-kk, and UNI and UNI+ in ru-en. These results suggest that QE systems such as UNI and UNI+ perform worse on judging systems of wide ranging quality, but better for top performing systems, or perhaps for systems closer in quality.

If our method of human assessment is sound, we should believe that BLEU, a widely applied metric, is no longer a reliable metric for judging our best systems. Future investigations are needed to understand when BLEU applies well, and why BLEU is not effective for output from our state of the art models.

Metrics and QE systems such as BERTr, ESIM, YiSi that perform well at judging our best systems often use more semantic features compared to our n-gram/char-gram based baselines. Future metrics may want to explore a) whether semantic features such as contextual word embeddings are achieving semantic understanding and b) whether semantic understanding is the true source of a metric’s performance gains.

It should be noted that some language pairs do not show the strong degrading pattern with top-n systems this year, for instance en-cs, engu, en-ru, or kk-en. English-Chinese is particularly interesting because we see a clear trend towards better correlations as we reduce the set of underlying systems to the top scoring ones.

## 6.2 Overall Metric Performance

## 6.2.1 System-Level Evaluation

In system-level evaluation, the series of YiSi metrics achieve the highest correlations in several language pairs and it is not significantly outperformed by any other metrics (denoted as a “win” in the following) for almost all language pairs.

The new metric ESIM performs best on 5 language languages (18 language pairs) and obtains 11 “wins” out of 16 language pairs in which ESIM participated.

The metric EED performs better for language pairs out-of English and excluding English compared to into-English language pairs, achieving 7 out of 11 “wins” there.

<span id="page-18-0"></span>18
## 6.2.2 Segment-Level Evaluation

For segment-level evaluation, most language pairs are quite discerning, with only one or two metrics taking the “winner” position (of not being significantly surpassed by others). Only French-German differs, with all metrics performing similarly except the significantly worse sentBLEU.

YiSi-1\_srl stands out as the “winner” for all language pairs in which it participated. The excluded language pairs were probably due to the lack of semantic information required by YiSi-1\_srl. YiSi-1 participated all language pairs and its correlations are comparable with those of YiSi-1\_srl.

ESIM obtain 6 “winners” out of all 18 languages pairs.

Both YiSi and ESIM are based on neural networks (YiSi via word and phrase embeddings, as well as other types of available resources, ESIM via sentence embeddings). This is a confirmation of a trend observed last year.

## 6.2.3 QE Systems as Metrics

Generally, correlations for the standard reference-based metrics are obviously better than those in “QE as a Metric” track, both when using monolingual and bilingual golden truth.

In system-level evaluation, correlations for “QE as a Metric” range from 0.028 to 0.947 across all language pairs and all metrics but they are very unstable. Even for a single metric, take UNI for example, the correlations range from 0.028 to 0.930 across language pairs.

In segment-level evaluation, correlations for QE metrics range from -0.153 to 0.351 across all language pairs and show the same instability across language pairs for a given metric.

In either case, we do not see any pattern that could explain the behaviour, e.g. whether the manual evaluation was monolingual or bilingual, or the characteristics of the given language pair.

## 6.3 Dependence on Implementation

As it already happened in the past, we had multiple implementations for some metrics, BLEU and chrF in particular.

The detailed configuration of BLEU and sacreBLEU-BLEU differ and hence their scores and correlation results are different.

chrF and sacreBLEU-chrF use the same parameters and should thus deliver the same scores but we still observe some differences, leading to different correlations. For instance for German-French Pearson correlation, chrF obtains 0.931 (no win) but sacreBLEUchrF reaches 0.952, tying for a win with other metrics.

We thus fully support the call for clarity by Post (2018b) and invite authors of metrics to include their implementations either in Moses scorer or sacreBLEU to achieve a long-term assessment of their metric.

## 7 Conclusion

This paper summarizes the results of WMT19 shared task in machine translation evaluation, the Metrics Shared Task. Participating metrics were evaluated in terms of their correlation with human judgement at the level of the whole test set (system-level evaluation), as well as at the level of individual sentences (segment-level evaluation).

We reported scores for standard metrics requiring the reference as well as quality estimation systems which took part in the track “QE as a metric”, joint with the Quality Estimation task.

For system-level, best metrics reach over 0.95 Pearson correlation or better across several language pairs. As expected, QE systems are visibly in all language pairs but they can also reach high system-level correlations, up to .947 (Chinese-English) or .936 (English-German) by YiSi-1\_srl or over .9 for multiple language pairs by UNI.

An important caveat is that the correlations are heavily affected by the underlying set of MT systems. We explored this by reducing the set of systems to top-n ones for various ns and found out that for many language pairs, system-level correlations are much worse when based on only the better performing systems. With both good and bad MT systems participating in the news task, the metrics results can be overly optimistic compared to what we get when evaluating state-of-the-art systems.

<span id="page-19-0"></span>19
In terms of segment-level Kendall’s τ results, the standard metrics correlations varied between 0.03 and 0.59, and QE systems obtained even negative correlations.

The results confirm the observation from the last year, namely metrics based on word or sentence-level embeddings (YiSi and ESIM), achieve the highest performance.

## Acknowledgments

Results in this shared task would not be possible without tight collaboration with organizers of the WMT News Translation Task. We would like to thank Marcin Junczys-Dowmunt for the suggestion to examine metrics performance across varying subsets of MT systems, as we did in Appendix A.

This study was supported in parts by the grants 19-26934X (NEUREM3) of the Czech Science Foundation, ADAPT Centre for Digital Content Technology (www.adaptcentre. ie) at Dublin City University funded under the SFI Research Centres Programme (Grant 13/RC/2106) co-funded under the European Regional Development Fund, and Charles University Research Programme “Progres” Q18+Q48.

## References

- Colin Bannard and Chris Callison-Burch. 2005. Paraphrasing with bilingual parallel corpora. In Proceedings of the 43rd Annual Meeting on Association for Computational Linguistics, ACL ’05, pages 597–604, Stroudsburg, PA, USA. Association for Computational Linguistics.
- Loïc Barrault, Ondřej Bojar, Marta R. Costajussà, Christian Federmann, Mark Fishel, Yvette Graham, Barry Haddow, Matthias Huck, Philipp Koehn, Shervin Malmasi, Christof Monz, Mathias Müller, Santanu Pal, Matt Post, and Marcos Zampieri. 2019. Findings of the 2019 Conference on Machine Translation (WMT19). In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Ondřej Bojar, Christian Federmann, Barry Haddow, Philipp Koehn, Matt Post, and Lucia Specia. 2016. Ten Years of WMT Evaluation Campaigns: Lessons Learnt. In Proceedings of the LREC 2016 Workshop “Translation Evaluation
- – From Fragmented Tools and Data Sets to an Integrated Ecosystem”, pages 27–34, Portorose, Slovenia.
- Ondřej Bojar, Yvette Graham, and Amir Kamran. 2017. Results of the WMT17 metrics shared task. In Proceedings of the Second Conference on Machine Translation, Volume 2: Shared Tasks Papers, Copenhagen, Denmark. Association for Computational Linguistics.
- Qian Chen, Xiaodan Zhu, Zhen-Hua Ling, Si Wei, Hui Jiang, and Diana Inkpen. 2017. Enhanced lstm for natural language inference. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 1657–1668.
- Julian Chow, Pranava Madhyastha, and Lucia Specia. 2019a. Wmdo: Fluency-based word mover’s distance for machine translation evaluation. In Proceedings of Fourth Conference on Machine Translation.
- Julian Chow, Lucia Specia, and Pranava Madhyastha. 2019b. WMDO: Fluency-based Word Mover’s Distance for Machine Translation Evaluation. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Michael Denkowski and Alon Lavie. 2014. Meteor Universal: Language Specific Translation Evaluation for Any Target Language. In Proceedings of the Ninth Workshop on Statistical Machine Translation, pages 376–380, Baltimore, Maryland, USA. Association for Computational Linguistics.
- Jacob Devlin, Ming-Wei Chang, Kenton Lee, and Kristina Toutanova. 2019. BERT: Pre-training of deep bidirectional transformers for language understanding. In Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, Volume 1 (Long and Short Papers), pages 4171–4186, Minneapolis, Minnesota. Association for Computational Linguistics.
- George Doddington. 2002. Automatic Evaluation of Machine Translation Quality Using Ngram Co-occurrence Statistics. In Proceedings of the Second International Conference on Human Language Technology Research, HLT ’02, pages 138–145, San Francisco, CA, USA. Morgan Kaufmann Publishers Inc.
- William B. Dolan and Chris Brockett. 2005. Automatically constructing a corpus of sentential paraphrases. In Proceedings of the Third International Workshop on Paraphrasing (IWP2005).
- Erick Fonseca, Lisa Yankovskaya, André F. T. Martins, Mark Fishel, and Christian Federmann. 2019. Findings of the WMT 2019 Shared

<span id="page-20-0"></span>20
- Task on Quality Estimation. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Yvette Graham and Timothy Baldwin. 2014. Testing for Significance of Increased Correlation with Human Judgment. In Proceedings of the 2014 Conference on Empirical Methods in Natural Language Processing (EMNLP), pages 172–176, Doha, Qatar. Association for Computational Linguistics.
- Yvette Graham, Timothy Baldwin, Alistair Moffat, and Justin Zobel. 2013. Continuous Measurement Scales in Human Evaluation of Machine Translation. In Proceedings of the 7th Linguistic Annotation Workshop & Interoperability with Discourse, pages 33–41, Sofia, Bulgaria. Association for Computational Linguistics.
- Yvette Graham, Timothy Baldwin, Alistair Moffat, and Justin Zobel. 2014a. Is Machine Translation Getting Better over Time? In Proceedings of the 14th Conference of the European Chapter of the Association for Computational Linguistics, pages 443–451, Gothenburg, Sweden. Association for Computational Linguistics.
- Yvette Graham, Timothy Baldwin, Alistair Moffat, and Justin Zobel. 2016. Can machine translation systems be evaluated by the crowd alone. Natural Language Engineering, FirstView:1–28.
- Yvette Graham and Qun Liu. 2016. Achieving Accurate Conclusions in Evaluation of Automatic Machine Translation Metrics. In Proceedings of the 15th Annual Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, San Diego, CA. Association for Computational Linguistics.
- Yvette Graham, Nitika Mathur, and Timothy Baldwin. 2014b. Randomized significance tests in machine translation. In Proceedings of the ACL 2014 Ninth Workshop on Statistical Machine Translation, pages 266–274. Association for Computational Linguistics.
- Yvette Graham, Nitika Mathur, and Timothy Baldwin. 2015. Accurate Evaluation of Segment-level Machine Translation Metrics. In Proceedings of the 2015 Conference of the North American Chapter of the Association for Computational Linguistics Human Language Technologies, Denver, Colorado.
- Yinuo Guo and Junfeng Hu. 2019. Meteor++ 2.0: Adopt Syntactic Level Paraphrase Knowledge into Machine Translation Evaluation. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Aaron L.-F. Han, Derek F. Wong, and Lidia S. Chao. 2012. Lepor: A robust evaluation metric for machine translation with augmented factors. In Proceedings of the 24th International Conference on Computational Linguistics (COLING 2012), pages 441–450. Association for Computational Linguistics.
- Aaron L.-F. Han, Derek F. Wong, Lidia S. Chao, Liangye He, Yi Lu, Junwen Xing, and Xiaodong Zeng. 2013. Language-independent model for machine translation evaluation with reinforced factors. In Machine Translation Summit XIV, pages 215–222. International Association for Machine Translation.
- Philipp Koehn. 2004. Statistical significance tests for machine translation evaluation. In Proc. of Empirical Methods in Natural Language Processing, pages 388–395, Barcelona, Spain. Association for Computational Linguistics.
- Philipp Koehn and Christof Monz. 2006. Manual and Automatic Evaluation of Machine Translation Between European Languages. In Proceedings of the Workshop on Statistical Machine Translation, StatMT ’06, pages 102–121, Stroudsburg, PA, USA. Association for Computational Linguistics.
- Gregor Leusch, Nicola Ueffing, and Hermann Ney. 2003. A novel string-to-string distance measure with applications to machine translation evaluation. In Proceedings of Mt Summit IX, pages 240–247.
- Gregor Leusch, Nicola Ueffing, and Hermann Ney. 2006. CDER: Efficient MT Evaluation Using Block Movements. In In Proceedings of EACL, pages 241–248.
- Chi-kiu Lo. 2019. YiSi - a Unified Semantic MT Quality Evaluation and Estimation Metric for Languages with Different Levels of Available Resources. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Qingsong Ma, Ondřej Bojar, and Yvette Graham. 2018. Results of the WMT18 metrics shared task: Both characters and embeddings achieve good performance. In Proceedings of the Third Conference on Machine Translation, Volume 2: Shared Task Papers, Brussels, Belgium. Association for Computational Linguistics.
- Matouš Macháček and Ondřej Bojar. 2014. Results of the WMT14 metrics shared task. In Proceedings of the Ninth Workshop on Statistical Machine Translation, pages 293–301, Baltimore, MD, USA. Association for Computational Linguistics.
- Matouš Macháček and Ondřej Bojar. 2013. Results of the WMT13 Metrics Shared Task. In Proceed-

<span id="page-21-0"></span>21
- ings of the Eighth Workshop on Statistical Machine Translation, pages 45–51, Sofia, Bulgaria. Association for Computational Linguistics.
- Nitika Mathur, Tim Baldwin, and Trevor Cohn. 2019. Putting evaluation in context: Contextual embeddings improve machine translation evaluation. In Proc. of ACL (short papers). To appear.
- Kishore Papineni, Salim Roukos, Todd Ward, and Wei-Jing Zhu. 2002. BLEU: A Method for Automatic Evaluation of Machine Translation. In Proceedings of the 40th Annual Meeting on Association for Computational Linguistics, ACL ’02, pages 311–318.
- Maja Popovic. 2012. Morpheme- and POS-based IBM1 and language model scores for translation quality estimation. In Proceedings of the Seventh Workshop on Statistical Machine Translation, WMT@NAACL-HLT 2012, June 7-8, 2012, Montréal, Canada, pages 133–137.
- Maja Popović. 2015. chrF: character n-gram Fscore for automatic MT evaluation. In Proceedings of the Tenth Workshop on Statistical Machine Translation, Lisboa, Portugal. Association for Computational Linguistics.
- Maja Popović. 2017. chrF++: words helping character n-grams. In Proceedings of the Second Conference on Machine Translation, Volume 2: Shared Tasks Papers, Copenhagen, Denmark. Association for Computational Linguistics.
- Matt Post. 2018a. A call for clarity in reporting BLEU scores. In Proceedings of the Third Conference on Machine Translation: Research Papers, pages 186–191, Belgium, Brussels. Association for Computational Linguistics.
- Matt Post. 2018b. A call for clarity in reporting bleu scores. In Proceedings of the Third Conference on Machine Translation, Belgium, Brussels. Association for Computational Linguistics.
- Matthew Snover, Bonnie Dorr, Richard Schwartz, Linnea Micciulla, and John Makhoul. 2006. A study of translation edit rate with targeted human annotation. In In Proceedings of Association for Machine Translation in the Americas, pages 223–231.
- Peter Stanchev, Weiyue Wang, and Hermann Ney. 2019. EED: Extended Edit Distance Measure for Machine Translation. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Miloš Stanojević and Khalil Sima’an. 2015. BEER 1.1: ILLC UvA submission to metrics and tuning task. In Proceedings of the Tenth Workshop on Statistical Machine Translation, Lisboa, Portugal. Association for Computational Linguistics.
- Weiyue Wang, Jan-Thorsten Peter, Hendrik Rosendahl, and Hermann Ney. 2016a. Character: Translation edit rate on character level. In ACL 2016 First Conference on Machine Translation, pages 505–510, Berlin, Germany.
- Weiyue Wang, Jan-Thorsten Peter, Hendrik Rosendahl, and Hermann Ney. 2016b. Charac-Ter: Translation Edit Rate on Character Level. In Proceedings of the First Conference on Machine Translation, Berlin, Germany. Association for Computational Linguistics.
- Evan James Williams. 1959. Regression analysis, volume 14. Wiley New York.
- Elizaveta Yankovskaya, Andre Tättar, and Mark Fishel. 2019. Quality Estimation and Translation Metrics via Pre-trained Word and Sentence Embeddings. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.
- Ryoma Yoshimura, Hiroki Shimanaka, Yukio Matsumura, Hayahide Yamagishi, and Mamoru Komachi. 2019. Filtering Pseudo-References by Paraphrasing for Automatic Evaluation of Machine Translation. In Proceedings of the Fourth Conference on Machine Translation, Florence, Italy. Association for Computational Linguistics.

<span id="page-22-0"></span>22

### Full Page Description (Page 22)

**Source:** `assets/_page_22_Asset_0.jpg`

**Generated:** 2026-05-30 21:49:36

---

The image contains a series of line graphs comparing different metrics across various systems for the language pair de-cs. The x-axis represents different metrics, while the y-axis shows the values for each system. The systems are labeled at the top of each graph, and the metrics are indicated at the bottom. The graphs display the performance of different systems, with some showing a consistent trend while others exhibit fluctuations. The presence of error bars indicates the variability or uncertainty in the performance metrics.


### Full Page Description (Page 22)

**Source:** `assets/_page_22_Asset_1.jpg`

**Generated:** 2026-05-30 21:50:02

---

The image contains a series of line graphs representing various metrics for different models or systems, likely in the context of machine translation or natural language processing. The x-axis appears to represent different configurations or settings, while the y-axis represents the performance metric values. Each graph corresponds to a specific model or system, such as BEER, BERTr, CharacTER, EED, ESIM, and others. The lines within each graph show the performance of these models under varying conditions, with some models showing more stable performance than others. The labels at the top of each graph indicate the specific metric being measured, such as BLEU, TER, and others.


### Full Page Description (Page 22)

**Source:** `assets/_page_22_Asset_2.jpg`

**Generated:** 2026-05-30 21:49:23

---

The image contains a series of line graphs, each representing different metrics or models in the context of machine translation or natural language processing. The x-axis appears to represent different configurations or parameters, while the y-axis likely represents performance metrics such as BLEU scores or other evaluation metrics. The graphs are labeled with various names such as "BEER," "CharacTER," "EED," "ESIM," and "LEPORa," among others. The lines in each graph show the performance of these models across different configurations, with some models showing consistent performance while others exhibit more variability. The presence of error bars suggests that the data is based on multiple trials or runs.

## A Correlations for Top-N Systems