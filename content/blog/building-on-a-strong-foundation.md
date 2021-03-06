+++
date = "2019-05-23"
publishdate = "2019-05-22"
title = "Building On a Strong Foundation"
author = "Cody Soyland"
author_twitter = "codysoyland"
author_img = "7"
image = "/img/blog/building-on-a-strong-foundation/banner.png"
overlay_color = "blue" # blue, green, or light
+++

In 2017, we [open-sourced](/blog/hello-world/) Pilosa with the goal of transforming the world of big data analytics using the power of bitmap indices and distributed computing. After two years of development, 38 releases, and tens of thousands of downloads, we couldn't be more proud of the work we've done.

Over that time, we've learned a few lessons. First, even as Pilosa is [easy to get started with](/docs/getting-started/), our users are busy professionals and are looking for a more turn-key solution that smoothly integrates with a variety of data sources. Second, some of our users have come up with crazy and innovative ways to operate directly on Pilosa's compressed bitmaps in-situ—unfortunately, without a common interface to this data, their solutions become expensive one-offs that are difficult to develop and maintain. We knew that we needed to expose the ability to load new algorithms dynamically which could operate directly on Pilosa's data plane.

Today, we are launching [Molecula](https://www.molecula.com/), a Data Virtualization platform built around Pilosa. Molecula is not just a new brand. It's a new identity for us, and a suite of products that furthers our original mission, making data instantly accessible to both humans and machines.

We are beyond excited for what we'll do with Molecula, but we aren't done with Pilosa. We are as committed as ever to maintaining and improving Pilosa, with many new features in active development, including new data types, better memory management, new query language features, and an algorithm extension framework. The open source Pilosa will be the foundation on which we build Molecula. Please have a look at our [new website](https://www.molecula.com/)!
