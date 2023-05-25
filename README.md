# Git-Repository
Recoding the voice sample from computer microphone
/** @file runmicrophone.c++
 *  @brief Test public functions from interfaceMicrophone.h
 *
 *  @author Atul kapade
 *  
 *  
 *  
 */

// -----------------
// standard includes
// -----------------
#include <cassert>
#include <numeric>
#include <vector>
#include <cmath>
#include <algorithm>

// ----------------
// library includes
// ----------------
#include "interfaceMicrophone.h"

// ------------
// captureAudio
// ------------

/**
 * @brief Attempt to listen on default recording device.
 *
 * @return true, if test passed.
 */
int captureAudio () {
	std::cerr << "**Running test captureAudio**" << std::endl;

	InterfaceMicrophone microphone;

	// Attempt to capture audio, status should be 1.
	int status = microphone.initFlow();
	bool retVal = false;
	if (status == 1) {
		retVal = true;
	}
	microphone.stopFlow();

	if (!retVal) {
		std::cerr << "!!Failed captureAudio test!!" << std::endl;
	} else {
		std::cerr << "--Passed--" << std::endl;
	}

	return retVal;
}

// ------------------
// captureAudioReInit
// ------------------

/**
 * @brief Attempt to re-initialize on a listener.
 *
 * @return true, if test passed.
 */
int captureAudioReInit () {
	std::cerr << "**Running test captureAudioReInit**" << std::endl;
	InterfaceMicrophone microphone;

	// Record audio samples for ~2s.
	int status = microphone.initFlow();
	Pa_Sleep(2*1000);

	// Attempt to re-initialize.
	status = microphone.initFlow();
	bool retVal = false;
	microphone.stopFlow();

	// Status should be 0 for a re-initialization.
	if (status == 0) {
		retVal = true;
	}

	if (!retVal) {
		std::cerr << "!!Failed captureAudioReInit test!!" << std::endl;
	} else {
		std::cerr << "--Passed--" << std::endl;
	}

	return retVal;
}

// ---------------
// appendDataValid
// ---------------

/**
 * @brief Attempt to get entropic bytes (recorded from a listener).
 *
 * @return true, if test passed.
 */
int appendDataValid () {
	std::cerr << "**Running test appendDataValid**" << std::endl;
	InterfaceMicrophone microphone;

	// Record audio samples for ~5s.
	microphone.initFlow();
	Pa_Sleep(5*1000);
	microphone.stopFlow();
	std::vector<uint8_t> data;

	// Append bytes recorded samples; sum should be non-zero.
	microphone.appendData(data);
	size_t sum = std::accumulate(data.begin(), data.end(), 0);
	bool retVal = (sum > 0);

	if (!retVal) {
		std::cerr << "!!Failed appendDataValid test!!" << std::endl;
	} else {
		std::cerr << "--Passed--" << std::endl;
	}

	return retVal;
}

// -------------------
// measureEntropyValid
// -------------------

/**
 * @brief Attempt to measure bit occurrence probabilities (entropy estimate) on
 *        recorded samples.
 * @return true, if test passed.
 */
int measureEntropyValid () {
	std::cerr << "**Running test measureEntropyValid**" << std::endl;

	InterfaceMicrophone microphone;

	// Record audio samples for ~10s.
	microphone.initFlow();
	Pa_Sleep(10*1000);
	microphone.stopFlow();

	// Compute mean bit entropy estimate over sample size.
	std::vector<double> entropy = microphone.bitEntropy();
	double normalizer = entropy.size();
	std::transform(
		entropy.begin(),
		entropy.end(),
		entropy.begin(),
		[normalizer] (double val) {
			return val / normalizer;
		}
	);

	// Check if mean entropy is atleast 0.1
	double meanEntropy = std::accumulate(entropy.begin(), entropy.end(), 0.0f);
	bool retVal = (meanEntropy > 0.1);

	if (!retVal) {
		std::cerr << "!!Failed measureEntropyValid test!!" << std::endl;
	} else {
		std::cerr << "--Passed--" << std::endl;
	}

	return retVal;
}

// -----------------
// appendDataInvalid
// -----------------

/**
 * @brief Attempt to get entropic bytes before initialization of a listening
 *        stream.
 * @return true, if test passed.
 */
int appendDataInvalid () {
	std::cerr << "**Running test appendDataInvalid**" << std::endl;

	InterfaceMicrophone microphone;
	std::vector<uint8_t> data;

	// Attempt to get data from microphone before recording samples.
	microphone.appendData(data);
	bool retVal = data.empty();

	if (!retVal) {
		std::cerr << "!!Failed appendDataInvalid test!!" << std::endl;
	} else {
		std::cerr << "--Passed--" << std::endl;
	}

	return retVal;
}

// ---------------------
// measureEntropyInvalid
// ---------------------

/**
 * @brief Attempt to measure bit occurrence probabilities (entropy estimate)
 *        when entropic data is not available or has been flushed.
 * @return true, if test passed.
 */
int measureEntropyInvalid () {
	std::cerr << "**Running test measureEntropyInvalid**" << std::endl;

	InterfaceMicrophone microphone;
	bool retVal;
	std::vector<uint8_t> data;

	// Attempt to get entropy estimate before recording audio samples.
	std::vector<double> entropy = microphone.bitEntropy();
	double meanEntropy = std::accumulate(entropy.begin(), entropy.end(), 0.0f);
	retVal = (std::fabs(meanEntropy) < 0.01f);

	// Record audio samples for ~10s.
	microphone.initFlow();
	Pa_Sleep(10*1000);
	microphone.stopFlow();

	// Flush data.
	microphone.appendData(data);

	// Attempt to get entropy after flushing (mean should ~ 0).
	entropy = microphone.bitEntropy();
	meanEntropy = std::accumulate(entropy.begin(), entropy.end(), 0.0f);
	retVal = retVal && (std::fabs(meanEntropy) < 0.01f);

	if (!retVal) {
		std::cerr << "!!Failed measureEntropyInvalid test!!" << std::endl;
	} else {
		std::cerr << "--Passed--" << std::endl;
	}

	return retVal;
}


int main(){
	/* Run tests and count passed.
	 * Order matters.
	 */
	int passed = 0;
	passed += captureAudio();
	passed += captureAudioReInit();
	passed += appendDataValid();
	passed += measureEntropyValid();
	passed += appendDataInvalid();
	passed += measureEntropyInvalid();


	std::cerr << std::endl;
	std::cerr << "--Passed " << passed << "/6" << " tests--" << std::endl;

	// Assert passing all tests.
	assert(passed == 6);

	return 0;
}
