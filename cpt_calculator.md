
# Cumulative Prospect Levers

First, we set up the CPT equations for probability weighting, and value
transforms. While several weighting functions have been suggested, here I'm
going to stick to Tversky & Kahneman's original version.


        def weighting(probability, power):
            return pow(probability, power) / pow(pow(probability, power) + pow(1. - probability, power), 1. / power)
    
    
        def value(outcome, lmbda, alpha, beta):
            """
            Return the relative value of an outcome.
            """
            if outcome > 0:
                return gain_value(outcome, alpha)
            elif outcome < 0:
                return lmbda * loss_value(outcome, beta)
            return 0
    
        def gain_value(payoff, alpha):
            if alpha > 0:
                payoff = pow(payoff, alpha)
            elif alpha < 0:
                payoff = math.log(payoff)
            else:
                payoff = 1 - pow(1 + payoff, alpha)
            return payoff
    
        def loss_value(payoff, beta):
            if beta > 0:
                payoff = -pow(-payoff, beta)
            elif beta < 0:
                payoff = pow(1 - payoff, beta) - 1
            else:
                payoff = -math.log(-payoff)
            return payoff

Now, recall our three levers - red, blue, and purple.

Under prospect theory, each possible outcome of a lever pull, together with the
probability of that outcome is a prospect. So, the red and blue levers have two
prospects a piece, and the purple just one.


    red_lever = [(0.8, 7), (0.2, 0)]
    blue_lever = [(0.11, 100), (0.89, -5)]
    purple_lever = [(1, 6)]

CPT also has a number of parameters (there is substantial, and ongoing debate
about the right values), the values for which I've also taken directly from T&K.

* α (alpha), is the power for gains when transforming values.
* β (beta), is the same for losses.
* λ (lmbda), is the loss aversion multiplier.
* γ (gamma), weights probabilities for gains.
* δ (delta), does the same for loses.


    alpha = 0.88
    beta = 0.88
    lmbda = 2.25
    gamma = 0.61
    delta = 0.69

Now we can compute the weighted probabilities, and transformed values for the
prospects. We'll start with the first prospect for the red lever - an 80% chance
of winning €7. This is a gain, so we use α.


    probability = red_lever[0][0]
    weighted_prob = weighting(probability, gamma)
    weighted_prob




    0.6074392743239481



We can also calculate the transformed value.


    outcome = red_lever[0][1]
    transformed_outcome = value(outcome, lmbda, alpha, beta)
    transformed_outcome




    5.54225208209486



And together they give us the value of this prospect.


    red_lever_cpt_value_1 = transformed_outcome*weighted_prob
    red_lever_cpt_value_1




    3.366581582868092



To get the CPT value for pulling the lever, we also need to do this for the
other prospect (a 20% chance of getting nothing), and sum the results.


    red_lever_cpt_value = value(red_lever[1][1], lmbda, alpha, beta)*weighting(red_lever[1][0], gamma)+red_lever_cpt_value_1
    red_lever_cpt_value




    3.366581582868092



By contrast, a straight expected value calculation would give...


    red_lever[1][1]*red_lever[1][0] + red_lever[0][1]*red_lever[0][0]




    5.6000000000000005



If we follow the same procedure to get the CPT value for the other two levers,
we can then compare them, and make our choice.


    blue_lever_cpt_value = value(blue_lever[0][1], lmbda, alpha, beta)*weighting(blue_lever[0][0], gamma) + value(blue_lever[1][1], lmbda, alpha, beta)*weighting(blue_lever[1][0], delta)
    purple_lever_cpt_value = value(purple_lever[0][1], lmbda, alpha, beta)*weighting(purple_lever[0][0], gamma)
    
    print "Blue lever CPT value is %f\nRed lever CPT value is %f\nPurple lever CPT value is %f" % (blue_lever_cpt_value, red_lever_cpt_value, purple_lever_cpt_value)

    Blue lever CPT value is 4.162055
    Red lever CPT value is 3.366582
    Purple lever CPT value is 4.839195


So, a certain outcome is preferred, and we pull the purple lever.


    
