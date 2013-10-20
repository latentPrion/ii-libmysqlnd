<?php

class iimysqli_resultC
{
	public $stmt, $nCols, $metadata;
}

function iimysqli_stmt_get_result($stmt)
{
	/**	EXPLANATION:
	 * We are creating a fake "result" structure to enable us to have
	 * source-level equivalent syntax to a query executed via
	 * mysqli_query().
	 *
	 *	$stmt = mysqli_prepare($conn, "");
	 *	mysqli_bind_param($stmt, "types", ...);
	 *
	 *	$param1 = 0;
	 *	$param2 = 'foo';
	 *	$param3 = 'bar';
	 *	mysqli_execute($stmt);
	 *	$result iimysqli_stmt_get_result($stmt);
	 *		[ $arr = iimysqli_result_fetch_array($result);
	 *			|| $assoc = iimysqli_result_fetch_assoc($result); ]
	 *	mysqli_stmt_close($stmt);
	 *	mysqli_close($conn);
	 *
	 * At the source level, there is no difference between this and mysqlnd.
	 **/
	$metadata = mysqli_stmt_result_metadata($stmt);
	$ret = new iimysqli_resultC;
	if (!$ret) return NULL;

	$ret->nCols = mysqli_num_fields($metadata);
	$ret->stmt = $stmt;

	mysqli_free_result($metadata);
	return $ret;
}

function iimysqli_result_num_rows($result)
{
	return mysqli_stmt_num_rows($result->stmt);
}

function iimysqli_result_data_seek($result, $offset)
{
	mysqli_stmt_data_seek($result->stmt, $offset);
}

function iimysqli_result_fetch_array(&$result)
{
	$ret = array();
	$code = "return mysqli_stmt_bind_result(\$result->stmt";

	if ($result->nCols <= 0) { return NULL; };

	for ($i=0; $i<$result->nCols; $i++)
	{
		$ret[$i] = NULL;
		$code .= ", \$ret['" .$i ."']";
	};

	$code .= ");";
	if (!eval($code)) { return NULL; };

	// This should advance the "$stmt" cursor.
	if (!mysqli_stmt_fetch($result->stmt)) { return NULL; };

	// Return the array we built.
	return $ret;
}

function iimysqli_result_fetch_assoc(&$result)
{
	$ret = array();
	$code = "return mysqli_stmt_bind_result(\$result->stmt";

	if ($result->nCols <= 0) { return NULL; };

	$metadata = mysqli_stmt_result_metadata($result->stmt);
	while ($field = mysqli_fetch_field($metadata))
	{
		$ret[$field->name] = NULL;
		$code .= ", \$ret['" .$field->name ."']";
	};

	mysqli_free_result($metadata);

	$code .= ");";
	if (!eval($code)) { return NULL; };

	// This should advance the "$stmt" cursor.
	if (!mysqli_stmt_fetch($result->stmt)) { return NULL; };

	// Return the array we built.
	return $ret;
}

?>
