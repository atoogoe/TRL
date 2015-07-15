# TRL
NAME         

        TRL - Program for transfer rule learning, variable selection and
        discretization, and data preprocessing.

SYNOPSYS
        java [JAVA_PARAMETERS] -jar TRL.jar
            [-lp LEARNING_PARAMETERS]
            [-ppp PREPROCESSING_PARAMETERS]
            -dp DATA_PARAMETERS

        java [JAVA_PARAMETERS] algorithm.main.TRL
            [-lp LEARNING_PARAMETERS]
            [-ppp PREPROCESSING_PARAMETES]
            -dp DATA_PARAMETERS

DESCRIPTION
                                        
    TRL is a classification learner that learns a rule-based classifier from 
    data. The program implements two overall algorithms for learning: classic 
    RL and Transfer RL (TRL). Both algorithms have many learning parameters 
    with sensible default values. The program can also transform the data in 
    various ways before learning, or even without any learning.

    TRL's input is a set of training data instances (examples), specified in a 
    data file (see DATA FILE FORMAT), where each instance is a vector of values 
    for the input variables, and a class value. The variables can be continuous 
    or categorical. The learned classifier comprises a set of rules of the 
    form:

    IF <antecedent> THEN <consequent>

    where the antecedent consists of a logical conjunction of one or more
    variable-value pairs (conditions), and the consequent is a prediction of the
    class variable.  For example, a learned rule might be:

    IF ((Age=High) AND (BloodPressure=Low)) THEN Class=Control

    which means "if the variable Age is in the High range, and the variable
    BloodPressure is in the Low range, then predict that the data
    instance has the class value Control". Values such as Low and High 
    represent intervals of real numbers that result from discretizing the 
    variables before training with TRL. A rule is said to cover or match a data
    instance if each variable value of the instance is in the range specified 
    in the rule antecedent. The classifier also includes an evidence gathering 
    method for breaking ties when several rules match a query data instance but
    predict different classes.

    The classic RL algorithm proceeds as a heuristic beam search through the 
    space of rules from general to specific. Starting with all rules containing 
    no variable-value pairs, it iteratively specializes the rules by adding
    conjuncts to the antecedent. It evaluates the rules, calculating a 
    certainty factor value and other statistics for each rule. It re-inserts 
    promising rules onto the beam, while removing other rules. The beam is 
    sorted by decreasing certainty factor value and is trimmed to a pre-defined
    length during each iteration. Beam search is used to limit the running time
    and space of the algorithm.

    Multiple learned rules in an RL classifier may cover the same training 
    instance. This is unlike most other classification rule and tree learning 
    algorithms, which cover data without replacement, so that each data 
    instance is covered by only one rule. With small sample size data sets, 
    covering with replacement allows TRL to utilize more of the available 
    evidence for each rule when computing the generalizability of the rule.  

    Classic RL also allows transfer rule learning (TRL), where some "prior" 
    rules are learned on a "source" data set and are then placed on the beam
    for learning on the "target" data set, while also learning new rules on
    this data set.
    
DATA FILE FORMAT

    Each data file comprises a table of rows (lines) and columns
    representing vectors of variable values. Example data file:

    #ID  Age  Sex  Temperature  @Diagnosis
    A42  22   F    37           Healthy
    D25  35   M    39           Sick
    ...  ...  ...  ...          ...
    
    In normally-oriented data files, each row represents a data instance
    vector and each column represents a variable (variable) of values for
    the instances; the first row is a header line specifying the names of the
    variables. However, the input file can be transposed, so that rows
    represent variables and columns represent data instances; this must be
    specified using the "-tpf" option.
    
    Columns can be separated by tabs or by commas (CSV). The separator must be
    the same throughout the file, and is assumed to be Tab unless "," occurs
    more frequently in the header line.  The user can explicitly specify
    the separator by using the data parameters.

    The data must contain exactly one class variable (output variable), and a 
    number of input variables. The class variable is indicated by a "@" as the 
    first character of the variable name. The class value of the first data 
    instance that appears in the file is used in the predictive performance 
    statistics in the output.

    A data file may contain one ID input variable, indicated by a "#" as the 
    first character of the variable name.  RL ignores the ID variable during 
    learning but uses it to identify data instances in the output. If no ID 
    variable is specified, the program uses data instance IDs "1", "2" and so 
    on, namely the index of the data instance in the data file.

    Each input variable can be continuous or categorical. A continuous variable
    is one whose values can be parsed as a numbers, such as "100" or
    "1.25" (without the quotes). Categorical variables have values such as "F" 
    and "M". before learning, TRL discretizes the continuous-valued variables 
    using the specified discretizer.  If you you want RL to treat some numeric 
    variable as categorical instead of continuous, and thus avoid discretizing 
    it, you can add a single quote before each value. For example, instead of 
    "100" (without the quotes), use "'100".

JAVA PARAMETERS

    Because TRL is a java program, it must be run on a Java virtual machine. 
    Java can take a number of parameters, which are described in the Java 
    manual. TRL uses some of the Weka program libraries, so Weka must be 
    installed on the system and the Java classpath must include the Weka 
    classes (jar file). The classpath can be set in the CLASSPATH environment 
    variable, or using the "-classpath" Java parameter. To provide enough 
    memory on the Java virtual machine, use the "-Xmx" Java parameter. For 
    example, "java -Xmx1000m".

TRL PARAMETERS

    Most TRL parameters have sensible default values that work well for most 
    data sets and so do not need to be set explicitly. See the EXAMPLES 
    section.

    Some parameters must be specified with a value. The value must be separated
    from the parameter flag by spaces. For example, "-cv 5", not "-cv5".

    TRL parameters are not case sensitive; for example, "-maxconj" means the
    same as "-MaxConj", "-MAXCONJ", etc.

LEARNING PARAMETERS

    Learning parameters affect the learning process. They can be specified in 
    any order after the Java parameters and before the "-ppp" flag. If no 
    learning parameters are specified, TRL will not do any learning, but will 
    only pre-process the input data. The "-lp" flag and the learning parameters
    must precede the pre-processing parameters and the data parameters in the 
    command line.

    When the '-tr' flags are not used, the program runs RL by default. With the
    '-tr' flag, we can invoke the TRL algorithms.

    -transferType INDEX_VAL
    -transferMethod INDEX_VAL
    -transfer INDEX_VAL
    -tr INDEX_VAL
            The type of transfer for prior rules. Prior rules are handled
            before any learning of new rules on the training data. The
            argument NUMBER can be:

            1   whole rules (default)
                Each prior rule is evaluated on the training data and put on
                the beam. To make sure that the source and training data have
                the same values for each variable, the source data is first 
                discretized using the training data discretization. Then
                
            2   rule structure
                Each prior rule is converted to a generalized structure where
                the variable values are removed, leaving only the variable
                in the LHS and the RHS. Then this structure is instantiated
                to create a set of rules with with all possible combinations
                values for the variable.

    -nocoverprior
    -nc
            Ignore the coverage of prior rules for inductive strengthening. 
            That is, examples covered by prior rules will be considered not 
            previously covered until they are covered by new rules.

    -nopriorrulesspecialize
    -nospecializeprior
            Do not specialize prior rules. If this option is omitted (default),
            prior rules are specialized on the beam.

    -cftype INDEX_VAL
            Function to compute the certainty factor value for each rule.
                0   Likelihood ratio
                1   Positive predictive value (default):
                        TP / (TP + FP)
                2   Positive predictive value with Yates correction:
                        (TP + 0.05) / (TP + FP)     if TP > FP,
                        (TP - 0.05) / (TP + FP)     if TP < FP,
                        TP / (TP + FP)              otherwise.
                3   Positive predictive value, normalized for asymmetric class
                    distributions:
                        1   if FP = 0
                        0   if TP + FP = 0
                        TP / (TP + FP * Pos / Neg)
                4   Laplace estimate:
                        (TP + 1) / (TP + FP + number_of_target_values + 1)
                5   Laplace extended:
                        (TP + k*m) / (TP + FP + k),
                    where
                        k = 1 + number of target values
                        m = Pos / (Pos + Neg)
                6   Laplace extended with bias for short rules:
                        (TP + c * q) / (TP + FP + k),
                    where
                        c = 1 + number of conjuncts in the rule
                        q = TP / (TP + TN)


    -inftype INDEX_VAL
            The "inference type" or "evidence-gathering" function that is to be
            used during inference to make a prediction from a given data
            instance from the learned set of rules. It combines the predictions
            of all rules that match the given instance.
                        
            0   Weighted voting (default). Predict the highest-weighted
                class, where the weight of each class is the sum of certainty
                factors of rules predicting that class. If there is a tie,
                predicts class 0.
            1   Maximum likelihood ratio
            2   "Combine CF"
            3   Lowest p-value: use the rule with the lowest p-value
            4   Single best: use the rule with the highest certainty factor
            5   Minimum weighted voting. Like weighted voting, but use only
                the highest k rules to calculate the weight of each class,
                where k is the minimum number of rules voting for any class.
            6   Single best specific: use the rule with the highest worth 
                (certainty divided by cost) and the highest number of cojuncts.
            7   Most specific single best: use the rule with the most conjuncts
                among rules with the highest certainty factor.

    -mincf NUMBER
            The minimum certainty factor value that any rule in the model
            will have.  The default is 0.85.

    -minconj NUMBER
            The minimum number of conjuncts in any rule in the model. The
            default is 1.

    -maxconj NUMBER
            The maximum number of conjuncts in any rule in the model. The
            default is 5.

    -specialize
            If this option is specified, when a rule is added to the model,
            RL will also check if some specializations of this rule should
            also be added to the model. If the option is omitted (default), 
            RL stops specialization of a rule once it is found to satisfy 
            the search constraints.

    -cover NUMBER
            The minimum number of training examples that any rule in the model 
            will cover. The default is 4.

    -minTP DECIMAL
            The minimum true positive rate that any rule in the model will
            have. The defalt is 0.05. Valid values are in the range [0, 1].

    -maxFP DECIMAL
            The maximum false positive rate that any rule in the model will
            have. This option is not set by default. Valid values are in the
            range [0, 1].
    
    -indStr NUMBER
            Inductive strengthening: the minimum number of previously uncovered
            examples that each new rule must cover. The default is 1. The
            smaller this number, the larger the overlap of instances covered by
            different rules. Because RL covers data with replacement, using some
            non-zero inductive strengthening helps to learn a more generalizable
            model.
            

    -beam WIDTH
            The number of rules kept at any time to be specialized in the next 
            iteration iteration. The default is 1000.

    -bss [--d DEPTH] [--f NUM_FOLDS]
            Do bias space search
         --d 0    Shallow search (default)
         --d 1    Medium depth search
         --d 2    Most exhaustive search; consumes a lot of time and memory
         --f NUM_FOLDS
                  Use internal cross-validation with NUM_FOLDS for validation

    -cv NUM_FOLDS
            Stratified cross-validation. If the option is not specified, no 
            cross-validation is performed. In any case, a classifier is 
            learned on all the training (target) data.

     
    -d      Discretize. The parameters are as described in
            PREPROCESSING PARAMETERS, but the discrete intervals for each
            variable are computed based on the training data only, then
            applied to the test data.  If cross-validation is specified, the
            discretization is computed on the training subset separately for
            each fold.

    -r      Remove trivial variables, as in PREPROCESSING PARAMETERS


PREPROCESSING PARAMETERS

    These are optional parameters that specify operations to be performed on 
    all the data before any rule learning. They can appear in any order after
    the "-lp" flag and before the "-dp" flag. Only operations specified will
    be performed.

    -d DISC_MECHOD DISC_VALUE
            Discretize using DISC_METHOD with specified parameter
            PARAMETER1  DISC_METHOD     PARAMETER2
                     0  GaussianU       Number of bins
                     1  EqualWidthU     Number of bins
                     2  EqualFreqU      Number of bins
                     3  OneR            Number of instances
                     4  ErrorBased      Max number of bins
                     5  D2S             (none - max number of bins is set to 8.)
                     6  FayyadIraniMDL  Number of bins
                     7  HEBD             c structure prior (use value "1")
                     8  MODL            none?
                     9  EBD             lambda prior 

            Example: EBD (2008) discretization with default parameter 1:
                -d 9 0.5

    -r      Remove trivial variables after discretization

    -chi NUM_OF_VARS_TO_SELECT 
            Chi-squared variables selection: select the top NUM_VARS_TO_SELECT 
            variables.

    -s SCALING_METHOD ...
            Scale each variable by the specified SCALING_METHOD in turn
            0   0-1 scaling
            1   Subtract local minimum
            2   Subtract global minimum
            3   Log2
            4   Square root
            5   Exponent 2
            6   Square
            7   Normalize to mean 0 and standard deviation 1

    -ctr
            Combine technical replicates. The samples must have the same name,
            with '#' next to it

DATA PARAMETERS

    Data parameters specify the input and output files and their format. The 
    training data file is a mandatory data parameter and must be the last 
    parameter. Data parameters must be preceded by the "-dp" flag. The "-dp" 
    flag and the data parameters must appear after the "-lp" flag and any 
    learning parameters, and after the "-ppp" flag and any pre-processing 
    parameters.

    -itrncsv
            Training data file is comma-separated

    -itstcsv
            Test data file is comma-separated
 
    -c CSV_DATA_FILE
            Convert the csv-delimited file to tab-delimited or vice versa.
     
    -tpf DATA_FILE
            Transpose the file data file; that is, make the rows be columns 
            (variables) and columns be rows (instances).

    -dtr TRAINING_DIRECTORY
            Directory containing the training data files, one file for each 
            training data instance. Each file contains two columns: variable
            and value. Within the directory files grouped by class folder; e.g.
            inside the training directory, there are two folders: "disease" and
            "control".  There should be no trailing "/" in TRAINING_DIRECTORY 
            name.

    -tst TEST_FILE
            Specify a test data file

    -dtst TEST_DIRECTORY
            Similar to -dtr.

    -od OUTPUT_DIRECTORY
            The output directory where to write the result files. The directory 
            is automatically created if it does not already exist.

    -o OUTPUT_DATA_FORMAT
            (arff - not implemented)
                WEKA's variable relation file format
            csv
                Comma-separated values format
            The default is tab-separated.

    -rand SEED
            Specifies a seed for creating random folds for running multiple 
            runs of RL with cross-validation. SEED is an integer. On Unix-like 
            systems and on Windows, a random integer is provided by the RANDOM 
            environment variable. If this option is not specified, the default
            seed is 1.

    -cmbf DIRECTORY
            Combine the files in DIRECTORY. Each file represents one training
            example (such as a mass spectrum), and contains two comma-
            separated columns. The first column contains the names of the 
            variables (such as M/Z values). The second column contains the 
            values for those variables (such as intensity values).

    -src
            Source data file for learning rules for transfer. These rules are
            put on the beam when learning on the target training data. If this
            option is not specified (default), RL does not do transfer learning.

            
    TRAINING_FILE
            The training data file is specified as the last argument. This
            argument must always be specified. In transfer algorithms, this
            data is the target dataset.


OUTPUT

    The program prints to standard output a log of its working that includes the 
    the program parameters (and learning parameters), clasifier learned,
    predictive performance statistics, starting time and total running time.
    
    For each rule, the log includes the following statistics: CF, CF/cost,
    p-value, true positive (TP) count, false positive (FP) count, and test TP
    and test FP.  These last two statistics are the number of test examples for
    which the rule was applied correctly (TP) or incorrectly (FP) when using the
    whole model. When applying the model, a rule may not fire even if it matches
    a test example, because of interaction with other rules. (See the discussion
    of evidence gathering under the "-inftype" parameter.)

    TRL also creates some output files containing any pre-processed data, rules
    learned on the whole training data set, rules learned from each cross-
    validation fold (if cross-validation was used), predictions on the data
    instances used in validation, and the predictive performance of the model
    calculated from the predictions. The files are in the output directory, which 
    by default is named as TRL_run_YYYY-MM-DD-hhmmss where the last part of the file
    name is the time when the TRL was run. Another output directory can be 
    specified using the "-od" parameter.

EXAMPLES

    Discretize the data using EBD, without learning:
        java -jar TRL.jar    -PPP -dr 9 0.5    -DP data.txt

    Learn using classic RL with default parameters (including EBD
    discretization) and 10-fold cross-validation.
        java -jar TRL.jar    -LP -cv 10    -PPP -dr 9 0.5    -DP data.txt

    10-fold cross-validation, EBD (2011) (specified explicitly), and classic RL
    with shallow (default) bias space search with 5-fold internal cross-
    validation with a random seed (on Unix):
        java -Xmx1300m -jar TRL.jar    -LP -rgm 0 -bss --d 0 --f 5 -cv 10 -d 7 1 -r  -PPP  -DP -rand $RANDOM data.txt

    Transfer learning with whole-rule transfer after averaging technical
    replicates and 10-fold cross-validation:
         java -Xmx1300m -jar TRL.jar    -LP -tr 2 -cv 10   -PPP -ctr  -DP -src source-data.txt target-data.txt


HISTORY

    The basic RL algorithm was described in Clearwater and Provost, 1990. It 
    was implemented in Lisp and then C in Bruce Buchanan's lab at the 
    University of Pittsburgh. The first Java version (which has a GUI) was 
    written in the same lab in the late 1990s by Jeremy Ludwig. It was modified
    and expanded by Joe Phillips 2001-2002, Will Bridewell 2001-2004, Eric 
    Williams 2002-2004, and Philip Ganchev 2002-2007 (dates are aproximate). 
    Mark Fenner wrote a command-line Python version during 2005-2007. The 
    current command-line version took much code from the GUI Java version.

    Jonathan Lustgarten 2006 -- 2009
    Philip Ganchev, March 2009 -- December 2010
    Jeya Balaji Balasubramanian, 2011 -- Present

PUBLICATIONS

    Clearwater S, Provost F. 1990. "RL4: a tool for knowledge-based induction."
    Proceedings fo the 2nd International IEEE Conference on Tools for 
    Artificial Intelligence.

    Gopalakrishnan V, Ganchev P, Ranganathan S, Bowser R. 2006. "Rule learning 
    for disease-specific biomarker discovery from clinical proteomic mass 
    spectra." Data Mining for Biomedical Applications.

    Kolli VSK, Seth B, Weaver L, Lustgarten JL, Gopalakrishnan V, Malehorn D, 
    Bigbee W, Mural RJ and Shriver CD. 2009. "Breast Cancer Sera for pattern 
    Analysis." Proceedings of the Human Protein Organisation Conference 2009.

    Lustgarten JL. 2009. "A Bayesian rule generation framework for 'omic' 
    biomedical data analysis." PhD thesis, University of Pittsburgh.

    Ganchev P. 2010. "Transfer rule learning for biomarker discovery and 
    verification from related data sets". PhD thesis, University of 
    Pittsburgh.

    Ranganathan S. 2004. "Amyotrophic lateral sclerosis molecular mechanisms to
    diagnosis." PhD thesis, University of Pittsburgh.

    Lacomis D, Urbinelli L, Newhall K, Cudkowicz ME, Brown RH Jr, Bowser R. 
    2005. "Proteomic profiling of cerebrospinal fluid identifies biomarkers of 
    amyotrophic lateral sclerosis." Journal of chemistry, 95(5):1461-1471.

    Ryberg H, An J, Darko S, Lustgarten J, Jaffa M, Gopalakrishnan V, 
    Lacomis D, Bowser R. 2010. "Discovery and verification of amyotrophic 
    lateral sclerosis biomarkers by proteomics." Muscle and Nerve, 
    41(1):104-111.

    Shortliffe EH, Buchanan BG. 1975. "A model of inexact reasoning in 
    medicine".  Mathematical Biosciences, 23(3-4):315-379.
