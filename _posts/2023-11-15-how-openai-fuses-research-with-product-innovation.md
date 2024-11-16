---
layout: post
title: "Bridging gaps: how OpenAI fuses research with product innovation"
date: 2023-11-15 08:00:00 +0000
categories: productmanagement, leadership
excerpt: "OpenAI's blend of AI research and product innovation, and how this approach is shaping the future of user-focused, effective AI solutions. And some of my own takes."
image: "/images/openai.png"
---

Keywords: AI, startup, research, product development, machine learning, OpenAI

  

## Video: How OpenAI Fuses Research with Product Innovation

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/YXiRbRacTF0?si=VAqT3GKYm1rg1u2z" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

  

### Key take-aways

<ul>
    <li>What contributed to the popularity of ChatGPT?
        <ul>
            <li><strong>General capabilities</strong> of models, contrasting with traditional models that are highly task-specific</li>
            <li><strong>Simple interfaces</strong>. Meets users at their level of information processing and knowledge creation.</li>
        </ul>
    </li>
    <li>Research x Product collaboration
        <ul>
            <li>Direct feedback loop: Product provides a direct feedback loop for research, using UX features such as the thumbs up/thumbs down buttons.</li>
            <li>Importance of product input in making research more applicable and user-focused
                <ul>
                    <li>Work closely together, contrary to 'traditional' setups where ML engineers throw a model over the fence to product</li>
                </ul>
            </li>
        </ul>
    </li>
    <li>Defining success
        <ul>
            <li>There is lots of discussions around ambiguity.
                <ul>
                    <li>what if the model is worse in physics, but better in biology?</li>
                    <li>What users want is subjective</li>
                </ul>
            </li>
            <li>Technology driven - instead of focusing on the user problems. The capabilities of GPT didn't fit into the existing paradigms yet!</li>
        </ul>
    </li>
    <li>The future of foundational models
        <ul>
            <li>AI models will be personalized to you
                <ul>
                    <li>OpenAI already does this using custom instructions, different profiles using different use cases (i.e. using their GPTs)</li>
                </ul>
            </li>
            <li>Models become multi-modal, encompassing all types of data (sound, text, images).</li>
            <li>More interfaces, not only text, to meet people where they are at in processing and creating knowledge</li>
            <li>Smarter models. Expansion to be more helpful in other areas with harder tasks: mathematics, scientific discoveries</li>
        </ul>
    </li>
</ul>


### My two cents

In the usual setup, it is often seen that ML engineers would 'throw models over the fence' to product. However, it is important to incorporate this ML research process into your product development, so that you can iterate faster. Few additional points:
- Make use of product/UX to reinforce your ML feedback loop! User input is valuable, especially when you have limited access to data and not know quantitatively what your AI boundaries are. This brings your ML engineers closer to your users and gets them the information to understand the user problems with the current AI.
- Be focused on providing business value as soon as possible, and limit your research to the absolute necessities [^1]. From my experience, ML Engineers have the tendency to build the perfect AI, but depending on the scenario, 70% might be good enough. *('But Ruben, how?' I will show you an example of an 'AI PRD' and how that should be different from a software oriented PRD soon! Feel free to reach out.)*


Thinking about the future of foundational models, it will take some time before they are able to tackle issues with expert-level knowledge, as we are not there yet [^2]. In the meantime, it is important to keep focusing on tailor-made solutions (be it 'traditional' deep learning or finetuned LLM), and to make sure to understand its weaknesses and strenghts. More often, most problems could be solved by wrapping some software functionalities around the model, catching and dealing with the AI weaknesses, resulting in an overall better performance and user experience. 



[^1]: Promaton has a very nice read-up on this, definitely check it out: [From zero to one: 7 essentials for successfully building and launching a medical AI product](https://blog.promaton.com/from-zero-to-one-7-essentials-for-successfully-building-and-launching-a-medical-ai-product-89adc783370a)


[^2]: In the healthcare domain, specific smaller tailored (BERT) models are still outperforming large language models: [Large language models in biomedical natural language processing: benchmarks, baselines, and recommendations](https://arxiv.org/ftp/arxiv/papers/2305/2305.16326.pdf)


---
