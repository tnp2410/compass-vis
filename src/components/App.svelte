<main>
  <h1>Restaurants Reviews Summary</h1>

  <p>D3 Visualization</p>

  <div id="word-cloud"></div>
</main>

<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import cloud from 'd3-cloud';

  // Use onMount to load data when the component mounts
  onMount(async () => {
    try {
      // Load JSON data using D3.json
      const data = await d3.json('example_output.json');
      console.log(data);

      // Process the reviews data into a format suitable for the word cloud
      const words = processReviews(data);

      // Generate word cloud layout
      const layout = cloud()
        .size([800, 600]) // Set the size of the word cloud layout
        .words(words)
        .padding(5) // Set padding between words
        .fontSize(d => Math.sqrt(d.value) * 10) // Scale font size based on list length (sqrt for better scaling)
        .rotate(() => (Math.random() > 0.5 ? 0 : 90)) // Randomly rotate words
        .on('end', drawWordCloud); // Callback to draw word cloud after layout is computed

      // Start the word cloud layout computation
      layout.start();

      // Helper function to process reviews into word data for the word cloud
      function processReviews(data) {
        const words = [];
        for (const key in data) {
          const reviews = data[key];
          const value = reviews.length; // Length of the reviews list
          words.push({ text: key, value: value });
        }
        return words;
      }

      // Callback function to draw the word cloud
      function drawWordCloud(words) {
        const svg = d3.select('#word-cloud')
          .append('svg')
          .attr('width', layout.size()[0])
          .attr('height', layout.size()[1])
          .append('g')
          .attr('transform', `translate(${layout.size()[0] / 2},${layout.size()[1] / 2})`);

        svg.selectAll('text')
          .data(words)
          .enter()
          .append('text')
          .style('font-size', d => `${Math.sqrt(d.value) * 10}px`) // Scale font size based on list length
          .style('fill', 'steelblue')
          .attr('text-anchor', 'middle')
          .attr('transform', d => `translate(${d.x},${d.y}) rotate(${d.rotate})`)
          .text(d => d.text)
          .on('mouseover', handleMouseOver)
          .on('mouseout', handleMouseOut);
        
        // Function to handle mouseover event
        function handleMouseOver(event, d) {
          d3.select(this)
            .style('fill', 'orange')
            .style('cursor', 'pointer');
        }

        // Function to handle mouseout event
        function handleMouseOut(event, d) {
          d3.select(this)
            .style('fill', 'steelblue');
        }
      }

    } catch (error) {
      console.error('Error loading data:', error);
    }
  });
</script>

<style>
  /* Write your CSS here */
  #word-cloud {
    text-align: center;
    margin-top: 20px;
  }
</style>
