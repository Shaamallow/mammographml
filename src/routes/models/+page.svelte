<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	let loaded = false;
	let divConfusionMatrix: HTMLDivElement;
	let divROC: HTMLDivElement;
	let divClassificationReport: HTMLDivElement;
	let divDecisionBoundaries: HTMLDivElement;

	let currentModel: string = 'No model selected';

	// prediction data
	let dataMLP: any;
	let dataSVM: any;
	let dataXGBoost: any;
	let dataLogistic: any;
	let dataReference: any;

	// classification report data
	let dataClassificationReport: any;
	let dataConfusionMatrix: any;

	let accuracy: number = 0;

	onMount(() => {
		loaded = true; // this is for the fade transition

		const parseValuesToFloat = (obj: Record<string, string | number>): Record<string, number> => {
			const result: Record<string, number> = {};

			for (const key in obj) {
				if (obj.hasOwnProperty(key)) {
					// Parse the value to float if it's a string
					result[key] =
						typeof obj[key] === 'string' ? parseFloat(obj[key] as string) : (obj[key] as number);
				}
			}

			return result;
		};

		async function loadPrediction(path: string) {
			const data = await d3.csv(path);

			let tempData = data.map((d) => parseValuesToFloat(d));
			let result: any = [];
			for (let i = 0; i < tempData.length; i++) {
				let row = tempData[i];
				result.push({
					id: i,
					prediction: row.predicted_diagnosis
				});
			}
			result.columns = ['id', 'prediction'];
			return result;
		}

		async function loadReference(path: string) {
			const data = await d3.csv(path);
			let temp = data.map((d) => parseValuesToFloat(d));
			let result: any = [];
			for (let i = 0; i < temp.length; i++) {
				let row = temp[i];
				result.push({
					id: i,
					PC1: row.PC1,
					PC2: row.PC2,
					diagnosis: row.real_diagnosis
				});
			}
			result.columns = ['id', 'PC1', 'PC2', 'diagnosis'];
			return result;
		}

		async function loadClassificationReport(path: string) {
			const data = await d3.csv(path);
			return data;
		}
		async function mergeData(data: any, reference: any) {
			let result: any = [];
			for (let i = 0; i < data.length; i++) {
				let row = data[i];
				result.push({
					id: i,
					PC1: reference[i].PC1,
					PC2: reference[i].PC2,
					diagnosis: reference[i].diagnosis,
					prediction: row.prediction
				});
			}
			result.columns = ['id', 'PC1', 'PC2', 'diagnosis', 'prediction'];
			return result;
		}

		const asyncLoad = async () => {
			dataLogistic = await loadPrediction('/predictions_log_reg.csv');
			dataSVM = await loadPrediction('/predictions_svm.csv');
			dataXGBoost = await loadPrediction('/predictions_xgb.csv');
			dataMLP = await loadPrediction('/predictions_mlp.csv');
			dataReference = await loadReference('/dataset_references.csv');

			dataLogistic = await mergeData(dataLogistic, dataReference);
			dataSVM = await mergeData(dataSVM, dataReference);
			dataXGBoost = await mergeData(dataXGBoost, dataReference);
			dataMLP = await mergeData(dataMLP, dataReference);

			dataClassificationReport = await d3.csv('/classification_report_data.csv');
			dataConfusionMatrix = await d3.csv('/confusion_matrix_data.csv');
		};

		asyncLoad();

		window.addEventListener('resize', draw);
	});

	function draw(): void {
		if (currentModel === 'Logistic') {
			drawConfusionMatrix('log_reg');
			drawROC(dataLogistic);
			drawClassificationReport('log_reg');
			drawDecisionBoundaries(dataLogistic);
		} else if (currentModel === 'SVM') {
			drawConfusionMatrix('svm');
			drawROC(dataSVM);
			drawClassificationReport('svm');
			drawDecisionBoundaries(dataSVM);
		} else if (currentModel === 'XGBoost') {
			drawConfusionMatrix('xgb');
			drawROC(dataXGBoost);
			drawClassificationReport('xgb');
			drawDecisionBoundaries(dataXGBoost);
		} else if (currentModel === 'MLP') {
			drawConfusionMatrix('mlp');
			drawROC(dataMLP);
			drawClassificationReport('mlp');
			drawDecisionBoundaries(dataMLP);
		}
	}

	function drawROC(data: any): void {}
	function drawConfusionMatrix(modelName: string): void {
		divConfusionMatrix.innerHTML = '';

		let padding = 10;
		let width = divConfusionMatrix.clientWidth - padding * 2;
		let height = divConfusionMatrix.clientHeight - padding * 2;

		let margin = 65;

		let trueNegative = parseFloat(dataConfusionMatrix[0][`${modelName}`]);
		let falseNegative = parseFloat(dataConfusionMatrix[1][`${modelName}`]);
		let falsePositive = parseFloat(dataConfusionMatrix[2][`${modelName}`]);
		let truePositive = parseFloat(dataConfusionMatrix[3][`${modelName}`]);

		let min_value = d3.min([trueNegative, falseNegative, truePositive, falsePositive]) as number;
		let max_value = d3.max([trueNegative, falseNegative, truePositive, falsePositive]) as number;

		let colorScale = d3.scaleSequential(d3.interpolateBlues).domain([min_value, max_value]);

		let svg = d3
			.select(divConfusionMatrix)
			.append('svg')
			.attr('width', width)
			.attr('height', height);

		let widthRect = (width - margin * 2) / 2;
		let heightRect = (height - margin * 2) / 2;

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin - 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.8rem')
			.text('Bening');

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin - 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.8rem')
			.text('Malignant');

		svg
			.append('text')
			.attr('x', margin + widthRect)
			.attr('y', margin - 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.text('Ground Truth');

		// text on the left
		svg
			.append('text')
			.attr('x', margin / 2)
			.attr('y', margin + heightRect / 2)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.9rem')
			.text('Bening');

		svg
			.append('text')
			.attr('x', margin / 2)
			.attr('y', margin + heightRect + heightRect / 2)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.8rem')
			.text('Malignant');

		// draw rectangles

		svg
			.append('rect')
			.attr('x', margin)
			.attr('y', margin)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(trueNegative));

		svg
			.append('rect')
			.attr('x', margin + widthRect)
			.attr('y', margin)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(falsePositive));

		svg
			.append('rect')
			.attr('x', margin)
			.attr('y', margin + heightRect)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(falseNegative));

		svg
			.append('rect')
			.attr('x', margin + widthRect)
			.attr('y', margin + heightRect)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(truePositive));

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect / 2)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(trueNegative.toFixed(0));

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect / 2 + 20)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '0.8rem')
			.text('TN');

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect / 2)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'black')
			.attr('font-size', '1.5rem')
			.text(falseNegative.toFixed(0));

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect / 2 + 20)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'black')
			.attr('font-size', '0.8rem')
			.text('FN');

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect + heightRect / 2)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'black')
			.attr('font-size', '1.5rem')
			.text(falsePositive.toFixed(0));

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect + heightRect / 2 + 20)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'black')
			.attr('font-size', '0.8rem')
			.text('FP');

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect + heightRect / 2)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(truePositive.toFixed(0));

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect + heightRect / 2 + 20)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '0.8rem')
			.text('TP');

		// title
		svg
			.append('text')
			.attr('x', '50%')
			.attr('y', margin / 3)
			.text('Confusion Matrix')
			.style('text-anchor', 'middle')
			.attr('class', 'font-display text-3xl font-bold fill-current');
	}
	function drawClassificationReport(modelName: string): void {
		divClassificationReport.innerHTML = '';

		let precisionRow = dataClassificationReport.filter((d: any) => d.Metrics == 'precision');
		let recallRow = dataClassificationReport.filter((d: any) => d.Metrics == 'recall');
		let f1Row = dataClassificationReport.filter((d: any) => d.Metrics == 'f1');
		let accuracyRow = dataClassificationReport.filter((d: any) => d.Metrics == 'accuracy');

		let precision = [
			parseFloat(precisionRow[0][`B_${modelName}`]),
			parseFloat(precisionRow[0][`M_${modelName}`])
		];
		let recall = [
			parseFloat(recallRow[0][`B_${modelName}`]),
			parseFloat(recallRow[0][`M_${modelName}`])
		];
		let f1 = [parseFloat(f1Row[0][`B_${modelName}`]), parseFloat(f1Row[0][`M_${modelName}`])];

		accuracy = parseFloat(accuracyRow[0][`B_${modelName}`]);

		let padding = 10;

		let width = divClassificationReport.clientWidth - padding * 2;
		let height = divClassificationReport.clientHeight - padding * 2;

		let margin = 80;

		let colorScale = d3.scaleSequential(d3.interpolateBlues).domain([0.8, 1]);

		let widthRect = (width - margin * 2) / 2;
		let heightRect = (height - margin * 2) / 3;

		let svg = d3
			.select(divClassificationReport)
			.append('svg')
			.attr('width', width)
			.attr('height', height);

		// add the 6 big rectangles for the classification report
		// add text on top of the rectangles for B and M
		// add legend on the left
		//
		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.text('Bening');

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.text('Malignant');

		// Precision on B
		svg
			.append('rect')
			.attr('x', margin)
			.attr('y', margin + 10)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(precision[0]));

		// Precision on M
		svg
			.append('rect')
			.attr('x', margin + widthRect)
			.attr('y', margin + 10)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(precision[1]));

		// Recall on B
		svg
			.append('rect')
			.attr('x', margin)
			.attr('y', margin + heightRect + 10)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(recall[0]));

		// Recall on M
		svg
			.append('rect')
			.attr('x', margin + widthRect)
			.attr('y', margin + heightRect + 10)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(recall[1]));

		// f1 on B
		svg
			.append('rect')
			.attr('x', margin)
			.attr('y', margin + heightRect * 2 + 10)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(f1[0]));

		// f1 on M
		svg
			.append('rect')
			.attr('x', margin + widthRect)
			.attr('y', margin + heightRect * 2 + 10)
			.attr('width', widthRect)
			.attr('height', heightRect)
			.attr('fill', colorScale(f1[1]));

		// white text only
		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(precision[0].toFixed(2));

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(precision[1].toFixed(2));

		svg
			.append('text')
			.attr('x', margin / 2)
			.attr('y', margin + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.9rem')
			.text('Precision');

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(recall[0].toFixed(2));

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(recall[1].toFixed(2));

		svg
			.append('text')
			.attr('x', margin / 2)
			.attr('y', margin + heightRect + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.9rem')
			.text('Recall');

		svg
			.append('text')
			.attr('x', margin + widthRect / 2)
			.attr('y', margin + heightRect * 2 + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(f1[0].toFixed(2));

		svg
			.append('text')
			.attr('x', margin + widthRect + widthRect / 2)
			.attr('y', margin + heightRect * 2 + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('fill', 'white')
			.attr('font-size', '1.5rem')
			.text(f1[1].toFixed(2));

		svg
			.append('text')
			.attr('x', margin / 2)
			.attr('y', margin + heightRect * 2 + heightRect / 2 + 10)
			.attr('text-anchor', 'middle')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current')
			.attr('font-size', '0.9rem')
			.text('F1');

		// add title
		svg
			.append('text')
			.attr('x', '50%')
			.attr('y', margin / 2)
			.text('Classification Report')
			.style('text-anchor', 'middle')
			.attr('class', 'font-display text-3xl font-bold fill-current');
	}
	function drawDecisionBoundaries(data: any): void {
		divDecisionBoundaries.innerHTML = ''; // equivalent of d3.select(divDecisionBoundaries).html(null) but in svelte;
		let padding = 10;

		// draw PCA prediction on div
		let width = divDecisionBoundaries.clientWidth - padding * 2;
		let height = divDecisionBoundaries.clientHeight - padding * 2;

		let margin = 50;

		let min_value_PC1 = d3.min(data, (d: any) => d.PC1 as number) as number;
		let max_value_PC1 = d3.max(data, (d: any) => d.PC1 as number) as number;

		const xScale = d3
			.scaleLinear()
			.domain([min_value_PC1 * 1.1, max_value_PC1 * 1.1])
			.range([margin, width - margin]);

		let min_value_PC2 = d3.min(data, (d: any) => d.PC2 as number) as number;
		let max_value_PC2 = d3.max(data, (d: any) => d.PC2 as number) as number;

		const yScale = d3
			.scaleLinear()
			.domain([min_value_PC2 * 1.1, max_value_PC2 * 1.1])
			.range([height - margin, margin]);

		const xAxis = d3.axisBottom(xScale);
		const yAxis = d3.axisLeft(yScale);

		const svg = d3
			.select(divDecisionBoundaries)
			.append('svg')
			.attr('width', width)
			.attr('height', height);

		// X axis
		svg
			.append('g')
			.attr('transform', `translate(0, ${height - margin})`)
			.call(xAxis)
			.append('text')
			.attr('x', width / 2)
			.attr('y', margin - 20)
			.attr('text-anchor', 'middle')
			.text('Principal Component 1')
			.attr('class', 'fill-current');

		// Y axis
		svg
			.append('g')
			.attr('transform', `translate(${margin}, 0)`)
			.call(yAxis)
			.append('text')
			.attr('x', -height / 2)
			.attr('y', -margin + 20)
			.attr('text-anchor', 'middle')
			.attr('transform', 'rotate(-90)')
			.attr('class', 'fill-current')
			.text('Principal Component 2');

		let dataTrueClassification = data.filter((d: any) => d.diagnosis == d.prediction);
		let dataFalseClassification = data.filter((d: any) => d.diagnosis != d.prediction);
		// Add dots
		let circles = svg
			.append('g')
			.selectAll('circle')
			.data(dataTrueClassification)
			.enter()
			.append('circle')
			.attr('cx', (d: any) => xScale(d.PC1))
			.attr('cy', (d: any) => yScale(d.PC2))
			.attr('r', 5)
			.attr('class', (d: any) => (d.diagnosis == 0 ? 'fill-primary ' : 'fill-error'))
			.attr('fill-opacity', 0.3);

		let rectangles = svg
			.append('g')
			.selectAll('rect')
			.data(dataFalseClassification)
			.enter()
			.append('rect')
			.attr('x', (d: any) => xScale(d.PC1) - 5)
			.attr('y', (d: any) => yScale(d.PC2) - 5)
			.attr('width', 10)
			.attr('height', 10)
			.attr('class', (d: any) => (d.diagnosis == 0 ? 'fill-primary ' : 'fill-error'))
			.attr('fill-opacity', 0.3);

		// add legend text on the right using a small circle and a text
		circles
			.on('mouseover', function (event: any, d: any) {
				let circlesWithSameDiagnostic = circles.filter((d2: any) => d2.diagnosis == d.diagnosis);

				circlesWithSameDiagnostic
					.attr('fill-opacity', 1)
					.attr('stroke', 'black')
					.attr('stroke-width', 1);

				d3.select(this).attr('r', 10);

				svg
					.append('text')
					.attr('x', xScale(d.PC1) + 10)
					.attr('y', yScale(d.PC2) - 10)
					.text(d.id)
					.attr('class', 'fill-current')
					.attr('id', 'tooltip');
			})
			.on('mouseout', function (event: any, d: any) {
				let circlesWithSameDiagnostic = circles.filter((d2: any) => d2.diagnosis == d.diagnosis);

				circlesWithSameDiagnostic.attr('fill-opacity', 0.3).attr('stroke', 'none');
				d3.select(this).attr('r', 5);
				svg.select('#tooltip').remove();
			});

		rectangles
			.on('mouseover', function (event: any, d: any) {
				let rectanglesWithSameDiagnostic = rectangles.filter(
					(d2: any) => d2.diagnosis == d.diagnosis
				);
				rectanglesWithSameDiagnostic
					.attr('fill-opacity', 1)
					.attr('stroke', 'black')
					.attr('stroke-width', 1);
				d3.select(this).attr('r', 10);
				svg
					.append('text')
					.attr('x', xScale(d.PC1) + 10)
					.attr('y', yScale(d.PC2) - 10)
					.text(d.id)
					.attr('class', 'fill-current')
					.attr('id', 'tooltip');
			})
			.on('mouseout', function (event: any, d: any) {
				let rectanglesWithSameDiagnostic = rectangles.filter(
					(d2: any) => d2.diagnosis == d.diagnosis
				);
				rectanglesWithSameDiagnostic.attr('fill-opacity', 0.3).attr('stroke', 'none');
				d3.select(this).attr('r', 5);
				svg.select('#tooltip').remove();
			});

		svg
			.append('circle')
			.attr('cx', width - margin * 2.5)
			.attr('cy', margin)
			.attr('r', 5)
			.attr('class', 'fill-primary');

		svg
			.append('rect')
			.attr('x', width - margin * 2.5 - 5)
			.attr('y', margin * 2.5 - 5)
			.attr('width', 10)
			.attr('height', 10)
			.attr('class', 'fill-primary');

		svg
			.append('circle')
			.attr('cx', width - margin * 2.5)
			.attr('cy', margin * 1.5)
			.attr('r', 5)
			.attr('class', 'fill-error');

		svg
			.append('rect')
			.attr('x', width - margin * 2.5 - 5)
			.attr('y', margin * 2 - 5)
			.attr('width', 10)
			.attr('height', 10)
			.attr('class', 'fill-error');

		svg
			.append('text')
			.attr('x', width - margin * 2.2)
			.attr('y', margin + 5)
			.text('TN')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'bottom')
			.attr('class', 'fill-current');
		svg
			.append('text')
			.attr('x', width - margin * 2.2)
			.attr('y', margin * 1.5 + 5)
			.text('TP')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');

		svg
			.append('text')
			.attr('x', width - margin * 2.2)
			.attr('y', margin * 2 + 5)
			.text('FN')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');

		svg
			.append('text')
			.attr('x', width - margin * 2.2)
			.attr('y', margin * 2.5 + 5)
			.text('FP')
			.style('font-size', '15px')
			.attr('alignment-baseline', 'middle')
			.attr('class', 'fill-current');

		// add title
		svg
			.append('text')
			.attr('x', '50%')
			.attr('y', margin / 2)
			.text('Predictions Test Set')
			.style('text-anchor', 'middle')
			.attr('class', 'font-display text-3xl font-bold fill-current');
	}

	function changeModel(e: Event): void {
		currentModel = (e.target as HTMLInputElement).value;
		draw();
	}
</script>

<svelte:head>
	<title>Models</title>
</svelte:head>

{#if loaded}
	<div in:fade={{ duration: 400, delay: 100 }} class="mx-auto transition duration-500">
		<div class="bg-base-100 py-10 px-4 mx-auto">
			<div
				class="flex flex-col items-center justify-stretch group max-w-3xl mx-auto overflow-visible"
			>
				<div class="md:flex block justify-center items-center md:mt-3 mb-6">
					<h1
						class="text-center font-display text-5xl md:text-7xl font-bold bg-clip-text bg-secondary anim-gradient pb-3"
					>
						Models
					</h1>
				</div>
				<div class="flex justify-center items-center mt-3 mb-6">
					<div class="font-display text-lg text-center">Breast Cancer Wisconsin Dataset</div>
				</div>
				<div class="text-center max-w-md">Classification of cells using different models</div>
			</div>
		</div>

		<!-- Model Selector -->

		<div class="container mx-auto flex flex-col lg:flex-row justify-center items-center gap-5 p-2">
			<input
				type="radio"
				name="model"
				class="btn hover:border-primary grow w-[13rem]"
				value="Logistic"
				aria-label="Logistic"
				on:change={changeModel}
			/>
			<input
				type="radio"
				name="model"
				class="btn hover:border-primary grow w-[13rem]"
				value="SVM"
				aria-label="Support Vector Machine"
				on:change={changeModel}
			/>
			<input
				type="radio"
				name="model"
				class="btn hover:border-primary grow w-[13rem]"
				value="XGBoost"
				aria-label="XGBoost"
				on:change={changeModel}
			/>
			<input
				type="radio"
				name="model"
				class="btn hover:border-primary grow w-[13rem]"
				value="MLP"
				aria-label="Multi-layer Perceptron"
				on:change={changeModel}
			/>
		</div>

		<div class="divider my-10" />

		<div class="mx-auto flex justify-center items-center mb-10">
			<h1 class="text-3xl">Accuracy : {accuracy}</h1>
		</div>

		<!-- Confusion and ROC -->
		<div
			class="container mx-auto flex flex-col lg:flex-row lg:flew-wrap justify-center items-center gap-5 p-2"
		>
			<div
				class="w-full h-[30rem] bg-neutral rounded-md p-2 pt-5"
				bind:this={divConfusionMatrix}
			></div>
			<div class="w-full h-[30rem] bg-neutral rounded-md p-2 pt-5" bind:this={divROC}></div>
		</div>

		<!-- Classification report and decision boundaries -->
		<div
			class="container mx-auto flex flex-col lg:flex-row lg:flew-wrap justify-center items-center gap-5 p-2"
		>
			<div
				class="w-full h-[30rem] bg-neutral rounded-md p-2 pt-5"
				bind:this={divClassificationReport}
			></div>
			<div
				class="w-full h-[30rem] bg-neutral rounded-md p-2 pt-5"
				bind:this={divDecisionBoundaries}
			></div>
		</div>

		<!-- PROJECTS FROM HERE -->
		<div class="divider m-10" />
	</div>
{/if}
